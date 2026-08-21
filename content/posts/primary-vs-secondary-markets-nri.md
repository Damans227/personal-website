---
title: "Primary vs Secondary Markets: Why NRIs Get an Extra Layer"
date: 2026-08-20T14:00:00Z
draft: false
tags:
  - Investing
  - NRI
  - India
  - Personal Finance
---

If you are an NRI who has tried to invest in India, you have probably hit a wall of confusing terms: PIS, non-PIS, NRE, NRO, primary, secondary. It feels like bureaucracy for its own sake. It is not. Underneath all of it sits one clean idea, with a monitoring layer stacked on top. Once you see the structure, the whole thing clicks.

---

## The one idea underneath everything: primary vs secondary

Every security lives in one of two markets, and the difference is simply who you are buying from.

**Primary market** is where a security is created and sold for the first time. Your money goes to the issuer, the company or the fund house, and they use it to fund the business or the fund. An IPO is the classic example: a company issues brand-new shares and raises capital. Buy a mutual fund and it is the same story. The AMC creates fresh units for you at NAV.

**Secondary market** is where securities that already exist change hands between investors. The issuer is not involved anymore. When you buy a listed stock on the exchange, your money goes to whoever sold it, not to the company. The company already got its money at the IPO. Everything after that is just investors passing ownership around.

A neat way to remember it: the issuer only gets paid once, at creation. Every trade after that is secondary.

Interestingly, the same security can live in both markets. A bond is primary when you buy it fresh from the issuer and secondary once it starts trading between investors. Only the first sale is primary.

---

## Why the division exists at all

This split is not an Indian invention. It exists in the US, Canada, Australia, the UK, everywhere. That is because it reflects two genuinely different economic jobs.

```
PRIMARY MARKET                      SECONDARY MARKET
--------------                      ----------------
Raises capital                      Transfers ownership
Issuer gets the money               Existing investors trade
Happens once per security           Happens continuously
Sets initial price                  Discovers real-time price
```

And here is the elegant part: they depend on each other. Nobody would buy into an IPO if they could never sell. The secondary market's whole purpose is to give investors an exit, and that liquidity is exactly what makes people willing to buy on the primary side in the first place. One feeds the other. The constant trading in the secondary market is also where a security's real-time price gets discovered, which in turn signals whether a company can cheaply raise more capital later.

---

## The part most investors never notice

If you are a resident, an Indian buying Indian stocks, or a Canadian buying Canadian stocks on Wealthsimple, you never think about primary vs secondary. You open one account and buy whatever you like: IPOs, funds, stocks, ETFs. The division still exists. It just runs invisibly in the background. You never have to act on it.

So why do NRIs suddenly have to care? Because of a monitoring layer that has nothing to do with market structure and everything to do with you being a non-resident.

---

## The RBI wrapper: PIS and non-PIS

When money crosses a country's border, governments want to watch it. Foreign capital flowing in and out moves the currency and can shake the economy, so central banks want real-time visibility. Foreign investors' gains also need correct taxing, often withheld at source, because the person lives abroad and is harder to chase later. Resident money staying inside the country is not a border event, so there is nothing to track. That is the entire reason residents skip this.

RBI's tool for watching NRI money is the **PIS (Portfolio Investment Scheme)** account. It maps onto the market structure like this:

- **Secondary market** (buying listed shares, ETFs) is routed through a **PIS account**, because share trading is what RBI most wants to monitor.
- **Primary market** (IPOs, mutual funds) runs through a **non-PIS NRE/NRO account**, no special surveillance channel needed.

One subtlety worth getting right: the market type comes first. It is just how securities work. PIS/non-PIS is layered on top as RBI's response. PIS does not create the structure. It attaches a bookkeeping wrapper to a structure that already exists. Market type is the reason. PIS/non-PIS is the paperwork around it.

And this is not unique to India. The US does the same thing to foreigners through W-8BEN forms and withholding tax on US dividends. Canada withholds tax on non-residents too. Every country layers extra compliance on foreign money. India just happens to use a named account rather than a form.

---

## NRE vs NRO: where the money comes from

Underneath the PIS layer are two bank account types, and they differ by the source of the money and whether you can take it back out.

**NRE (Non-Resident External)** holds foreign income you bring into India. It is fully repatriable, meaning you can convert back to foreign currency and move it all out anytime, and the interest is tax-free in India. Think: money from outside, freely flows back outside.

**NRO (Non-Resident Ordinary)** holds income earned inside India, like rent, dividends, capital gains, or a local pension. Repatriation is capped (roughly USD 1 million a year, with paperwork) and the gains are taxable, with TDS applied. Think: money generated in India, stays mostly in India.

Invest through NRE and your gains stay repatriable. Invest through NRO and they are stickier and taxed locally.

```
                  NRE                          NRO
                  ---                          ---
Money source      Foreign income               Indian income
                  (salary abroad, etc.)        (rent, dividends, pension)

Repatriable?      Yes, fully                   Capped (~USD 1M/year,
                                                with paperwork)

Interest tax      Tax-free in India            Taxable in India, TDS applies

Mental model      "Foreign money, free         "Indian money, stays mostly
                   to flow back out"            in India"
```

---

## What actually happens at the backend

Here is the part almost nobody explains: the PIS monitoring does not happen at your demat account. It happens at the **designated bank account** linked to your trading. The bank is RBI's reporting agent.

When you open a PIS account, a designated (RBI-authorized) bank issues you a permission letter and tags that account as PIS in the banking system. Every buy and sell of listed shares must route through it. On each trade:

1. The transaction flows through your tagged PIS account.
2. The designated bank sees it and reports it to RBI daily, showing how much came in on buys and how much went out on sells.
3. The bank enforces limits in real time, checking you have not breached NRI ownership caps (both individual and aggregate caps exist per company).
4. The bank withholds tax (TDS) on your capital gains at source before the money settles.

For non-PIS routes (IPOs, mutual funds through NRE/NRO), there is no per-transaction surveillance channel and no real-time ownership-cap policing. The money just moves through an ordinary account. Tax still applies (TDS on NRO gains) but the live monitoring layer is not there.

So the demat holds your shares, but the bank account is where the tracking, capping, and taxing actually happen.

---

## The whole thing on one page

```
                        WHAT YOU'RE BUYING
                                |
                +---------------+---------------+
                |                               |
          PRIMARY MARKET                 SECONDARY MARKET
      (created & sold fresh)          (existing, changes hands)
       IPOs, mutual funds              Listed shares, ETFs
                |                               |
                v                               v
       LAYER 2: RBI WRAPPER            LAYER 2: RBI WRAPPER
       Non-PIS                         PIS (monitored)
       (no surveillance)               (daily reporting +
                |                       cap checks + TDS)
                |                               |
                +---------------+---------------+
                                |
                                v
                    LAYER 3: BANK ACCOUNT
                       NRE or NRO
              +---------------+---------------+
              |                               |
             NRE                             NRO
     Foreign income in                Indian income in
     Fully repatriable                Cap ~USD 1M/year
     Tax-free interest                Taxed, TDS applies

The bank account is where reporting, cap enforcement,
and tax withholding actually happen. The demat just
holds the shares.
```

---

## The takeaway

Strip away the jargon and there are just three layers:

1. **Primary vs secondary** is universal market structure. Primary sells securities first-hand to investors. Secondary trades them between investors. Exists everywhere, invisible to residents.
2. **PIS vs non-PIS** is RBI's monitoring wrapper, applied only because you are a non-resident. Secondary goes through PIS. Primary goes through non-PIS.
3. **NRE vs NRO** are the bank accounts underneath, defined by where the money came from and whether you can take it back out.

A resident never meets any of this. The market structure is the reason. The rest is just a country keeping an eye on foreign money crossing its border, something every country does, one way or another.

---

*This is a general explainer, not tax or investment advice. NRI rules, caps, and TDS rates change. Confirm specifics with your bank or a qualified advisor.*
