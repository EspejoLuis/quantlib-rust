# November

## ☀️ 1 November 2025 - Saturday

- Checking the `ql` folder:
  - `cashflows`
  - `currencies`
  - `experimental`
  - `indexes`
  - `instruments`
  - `legacy`
  - `math`
  - `methods`
  - `models`
  - `patterns`
  - `pricingengines`
  - `processes`
  - `quotes`
  - `termstructures`
  - `time`
  - `utilities`
  - `loose section` i.e. other loose files

- Checking [Implementing Quantlib](https://leanpub.com/implementingquantlib) and other books from [Quantlib](https://www.implementingquantlib.com/) to understand better the structure of the library. I will try to do a summary of some points. Definitely better to download the book and check yourself as well.

  - The library has to be able to add *new financial instruments* and *new pricing engine* to the already existing financial instruments.
  
## Financial Instruments

- `instrument.cpp` contains the general interface of any instrument.

- Each instrument lives in its own world. Two instruments can be completely different. As a consequence, in `instrument.cpp`, there are going to be only essential methods that need to be passed to each instrument.

- **Two important requirements**:
  1. Instruments should invoke pricing engines without using inheritance.
  2. Instruments are linked to the market data and changing market data source should be treated as a change in market data value i.e. whether you change a value or source, the value of the instrument should be recomputed.
  Moreover, to avoid loss of permorce, Quantlib implements a cashing mechanism at instrument level.
     - Basically, market data can change:
       - In value
       - In source
     - If market data doesn't change, then use a cashed value.

## ☀️ 2 November 2025 - Sunday

### Financial Instruments - Implementation

- Useful link! : [Quantlib-Python Documentation](https://quantlib-python-docs.readthedocs.io/en/latest/indexes.html#histories)

- Compiled the quantlib documentation in the library i.e. `make docs`. Quantlib uses Doxygen. Was wondering if there is an equivalent in Rust. There is. It's rustdoc which is built in rust by defaults. It seems cool. To keep in mind.

- Link to the explanations by Ballabio: [Implementing Quantlib - Financial instruments](https://www.implementingquantlib.com/2013/07/chapter-2-part-1-of-4-financial.html)

- To undertand the implementation of the [second requirement](11_November.md#L36) we are goint to need following main components:
  - `instrument.cpp` is in `instruments` folder.
  - `lazyobject.cpp` is in `patterns`folder.
  - `handle.cpp` is in `ql` folder, in the `loose section`.

## ☀️ 4 November 2025 - Tuesday

### Financial Instruments - Implementation - Part II

- So there are two types of objects:
  - Observer -> Instrument (for example Bond).
  - Observable -> Data (for example YieldTermStructure).

- Instrument inherits `LazyObject` so it is an Observer (and also and Obeservable but see later). It has therefore an `update()` method that is called when an observable changes in value/source.

- How to implement a logic that allows to have an update when there is a change in value or in source ? by using a handle i.e. a pointer to a pointer. For example

    ```cpp
    Handle<Quote> rate;
    ```

  Why Handle is not just a pointer ?

    ```cpp
    shared_ptr<Quote> rate;
    ```

  A pointer would be useful in case where there is a change in value but not when there is a change in source:
  
  - If value changes, Quote changes and Handle notifies the observer.
  - If source changee, the entire object Quote is changed. If Instrument stores just pointer, the pointer will point to the old object anyway.

    ```cpp
    // Pointer - reacts to value change

    class Instrument {
        MarketData* data;
    public:
        Instrument(MarketData* d) : data(d) {}
        double NPV() { return 1000 / (1 + data->rate); }
    };

    MarketData feed(0.05);
    MarketData* activeFeed = &feed;
    Instrument bond(activeFeed);
    cout << bond.NPV(); // prints with 5%
    feed.rate = 0.04;
    cout << bond.NPV(); // prints with 4%

    // but switching the external pointer doesn't affect the bond
    MarketData otherFeed(0.03);
    activeFeed = &otherFeed;
    cout << bond.NPV(); // still prints with 4%
    ```

    ```cpp
    // Pointer-to-pointer - reacts to source change

    class Instrument {
        MarketData** handle;
    public:
        Instrument(MarketData** h) : handle(h) {}
        double NPV() { return 1000 / (1 + (*handle)->rate); }
    };

    MarketData bloomberg(0.05), reuters(0.04);
    MarketData* activeFeed = &bloomberg;
    MarketData** handle = &activeFeed;
    Instrument bond(handle);
    cout << bond.NPV(); // prints with 5%
    bloomberg.rate = 0.045;
    cout << bond.NPV(); // prints with 4.5%
    activeFeed = &reuters;
    cout << bond.NPV(); // now prints with 4%
    ```

## ☀️ 6 November 2025 - Thursday

### Financial Instruments - Implementation - Part III

- So with `handle`, instruments are updated if there is a change in value or in source. However, to improve performance the class `lazyObject` is used. This class takes care of two things:
  - *Lazyness* i.e. computes value only when necessary.
  - *Caching* i.e. store those values once computed.

- To be precise:
  - The computations (`performCalculations()`) has to be anyway implemented by derived classes.
  - Cashing is taking care by the base class `lazyObject` in the sense that the `calculated_` flag is set in the base. However, the value `NPV_` is store at instrument level. Also instrument has to decorate `calculate()`.

- In summary `lazyobject` has:
  - virtual `calculate()` -> const.
  - mutable bool `calculated_` : This is used in `calculate()`. It is mutable because when a new value is assigned to a data member, the object’s state is mutating! The C++ `const` contract is: no modification of object state in const methods. So `calculated_` needs to be declared mutable.
  - virtual `performCalculation()` -> const.
  - void `update()`. This set the `calculate_` flag to false.

- `Instrument` inherits from `lazyobject`:
  - real `NPV()` -> const
  - mutable `NPV_`. For same reason as before.
  - `calculate()` -> const. Decorates `calculate()` in `lazyobject` to check if the instrument is expired.

## ☀️ 7 November 2025 - Friday

### Financial Instruments - Implementation - Part IV

- To consolidate all the info from previous day, reading [IRS example](https://www.implementingquantlib.com/2013/07/chapter-2-part-2-of-4-example.html) is very useful. Expecially the class diagram of the Swap class.

- In the IRS, cash flows are pointers while term structure is a handle, because cash flows are inherently linked to the swap, they cannot be changed. On the other hand, you can use different curves to price the same swap.

- Important points so far (will tackled later)
  - How legs with different currencies are handled ?
  - Is there clear separation:
    - Data specifying the contract (info needed to define the cash flows)
    - Market data (discount cuve)

## Pricing Engines

Notes:

- Important points (from old version of the book, maybe now they are fixed):
  - How legs with different currencies are handled ?
  - Is there clear separation:
    - Data specifying the contract (info needed to define the cash flows)
    - Market data (discount cuve)