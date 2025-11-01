# ☀️ 1 November 2025

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
  - other loose files

- Checking [Implementing Quantlib](https://leanpub.com/implementingquantlib) and other books from [Quantlib](https://www.implementingquantlib.com/) to understand better the structure of the library. I will try to do a summary of some points. Definitely better to download the book and check yourself as well.

  - The library has to be able to add *new financial instruments* and *new pricing engine* to the already existing financial instruments.
  
## Financial Instruments

- `instrument.cpp` contains the general interface of any instrument.
- Each instrument lives in its own world. Two instruments can be completely different. As a consequence, in `instrument.cpp`, there are going to be only essential methods that need to be passed to each instrument.
- **Two important requirements**:
  1. Instruments should invoke pricing engines without using inheritance.
  2. Instruments are linked to the market data, and changing market data source should be treated as a change in market data value i.e. whether you change a value or source, the value of the instrument should be recomputed. One way to facilitate this is by implementing a cashing mechanism at instrument level.

## Pricing Engines
