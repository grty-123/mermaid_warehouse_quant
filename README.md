# 三层数据体系总览（可点击查看字段）

```mermaid
%%{init: {"securityLevel": "loose"}}%%
flowchart TD
  %% ---------- 数据源 ----------
  subgraph "数据源"
    ex["交易所(OKX/Binance/Bybit)"]
    cg["Coinglass API"]
    mys["MySQL 申赎/账变"]
    fxsrc["FX 汇率"]
    bench["基准/3M国债"]
  end

  %% ---------- 原始层 ODS ----------
  subgraph "原始层 ODS"
    inj["采集器(Python/cron 或 Kafka)"]
    raw_trades["ods.raw_trades"]
    raw_orders["ods.raw_orders"]
    raw_fills["ods.raw_fills"]
    raw_pos["ods.raw_positions"]
    raw_equity["ods.raw_equity"]
    raw_transfer["ods.raw_transfer"]
    raw_funding["ods.raw_funding_rate"]
    raw_ohlcv["ods.raw_ohlcv_1m"]
    raw_fx["ods.raw_fx_rate"]
    raw_bench["ods.raw_benchmark_nav"]
  end

  ex --> inj
  cg --> inj
  mys --> inj
  fxsrc --> inj
  bench --> inj

  inj --> raw_trades
  inj --> raw_orders
  inj --> raw_fills
  inj --> raw_pos
  inj --> raw_equity
  inj --> raw_transfer
  inj --> raw_funding
  inj --> raw_ohlcv
  inj --> raw_fx
  inj --> raw_bench

  %% ---------- 数仓层 DW ----------
  subgraph "数仓层 DW (ClickHouse + dbt)"
    direction TB
    subgraph "统一维度 DIM"
      dim_acct["dim_account"]:::tbl
      dim_asset["dim_asset"]:::tbl
      dim_instr["dim_instrument"]:::tbl
      dim_fx["dim_fx_rate"]:::tbl
      dim_bench["dim_benchmark"]:::tbl
    end

    subgraph "标准明细 DWD"
      f_trade["dwd.fct_trade"]:::tbl
      f_order["dwd.fct_order"]:::tbl
      f_fill["dwd.fct_fill"]:::tbl
      f_pos["dwd.fct_position_snapshot"]:::tbl
      f_equity["dwd.fct_equity_snapshot"]:::tbl
      f_transfer["dwd.fct_transfer"]:::tbl
      f_ledger["dwd.fct_ledger_entry"]:::tbl
      f_funding["dwd.fct_funding_rate"]:::tbl
      f_pay["dwd.fct_funding_payment"]:::tbl
      f_ohlcv["dwd.fct_ohlcv_1m"]:::tbl
      f_bench["dwd.fct_benchmark_nav"]:::tbl
    end

    subgraph "主题汇总 DWS"
      d_account["dws.account_day"]:::tbl
      d_lev["dws.leverage_day"]:::tbl
      d_fund_nav["dws.fund_nav_day"]:::tbl
      d_alpha["dws.alpha_day"]:::tbl
      d_ratio["dws.asset_ratio_day"]:::tbl
      d_exec["dws.exec_quality_minute"]:::tbl
      d_risk["dws.risk_events"]:::tbl
    end
  end

  %% ODS -> DWD
  raw_trades --> f_trade
  raw_orders --> f_order
  raw_fills --> f_fill
  raw_pos --> f_pos
  raw_equity --> f_equity
  raw_transfer --> f_transfer
  raw_funding --> f_funding
  raw_ohlcv --> f_ohlcv
  raw_fx --> dim_fx
  raw_bench --> f_bench

  %% DWD -> DWS
  f_equity --> d_account
  f_transfer --> d_account
  f_ledger --> d_account
  f_pos --> d_lev
  f_ohlcv --> d_lev
  d_account --> d_fund_nav
  d_fund_nav --> d_alpha
  f_bench --> d_alpha
  d_account --> d_ratio
  dim_acct --> d_ratio
  f_funding --> d_risk
  f_pay --> d_account

  %% ---------- 应用层 ADS ----------
  subgraph "应用层 ADS / 语义层"
    vw_port["ads.vw_portfolio_latest"]
    vw_nav["ads.vw_nav_daily"]
    vw_recon["ads.vw_equity_recon_diff"]
    graf["Grafana"]
  end

  d_fund_nav --> vw_nav
  d_alpha --> vw_port
  d_lev --> vw_port
  d_ratio --> vw_port
  d_account --> vw_port
  d_account --> vw_recon
  vw_port --> graf
  vw_nav --> graf
  vw_recon --> graf

  %% ---------- 点击跳转到字段详情（本页锚点） ----------
  click dim_acct   "#dim_account"         "查看 dim_account 字段"
  click dim_asset  "#dim_asset"          "查看 dim_asset 字段"
  click dim_instr  "#dim_instrument"     "查看 dim_instrument 字段"
  click dim_fx     "#dim_fx_rate"        "查看 dim_fx_rate 字段"
  click dim_bench  "#dim_benchmark"      "查看 dim_benchmark 字段"

  click f_trade    "#fct_trade"          "查看 dwd.fct_trade 字段"
  click f_order    "#fct_order"          "查看 dwd.fct_order 字段"
  click f_fill     "#fct_fill"           "查看 dwd.fct_fill 字段"
  click f_pos      "#fct_position_snapshot" "查看 dwd.fct_position_snapshot 字段"
  click f_equity   "#fct_equity_snapshot"   "查看 dwd.fct_equity_snapshot 字段"
  click f_transfer "#fct_transfer"       "查看 dwd.fct_transfer 字段"
  click f_ledger   "#fct_ledger_entry"   "查看 dwd.fct_ledger_entry 字段"
  click f_funding  "#fct_funding_rate"   "查看 dwd.fct_funding_rate 字段"
  click f_pay      "#fct_funding_payment" "查看 dwd.fct_funding_payment 字段"
  click f_ohlcv    "#fct_ohlcv_1m"       "查看 dwd.fct_ohlcv_1m 字段"
  click f_bench    "#fct_benchmark_nav"  "查看 dwd.fct_benchmark_nav 字段"

  click d_account  "#dws_account_day"    "查看 dws.account_day 字段"
  click d_lev      "#dws_leverage_day"   "查看 dws.leverage_day 字段"
  click d_fund_nav "#dws_fund_nav_day"   "查看 dws.fund_nav_day 字段"
  click d_alpha    "#dws_alpha_day"      "查看 dws.alpha_day 字段"
  click d_ratio    "#dws_asset_ratio_day" "查看 dws.asset_ratio_day 字段"
  click d_exec     "#dws_exec_quality_minute" "查看 dws.exec_quality_minute 字段"
  click d_risk     "#dws_risk_events"    "查看 dws.risk_events 字段"

  classDef tbl fill:#EEF5FF,stroke:#5B8FF9,stroke-width:1.2px;
