# 三层数据体系总览（GitHub 安全版：静态 mermaid）

```mermaid
flowchart TD
  %% ---------- 原始层 ----------
  subgraph ODS
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

  %% ---------- 数仓层 ----------
  subgraph DW["DW (ClickHouse + dbt)"]
    subgraph DIM
      dim_fx["dim_fx_rate"]
    end
    subgraph DWD
      f_trade["dwd.fct_trade"]
      f_order["dwd.fct_order"]
      f_fill["dwd.fct_fill"]
      f_pos["dwd.fct_position_snapshot"]
      f_equity["dwd.fct_equity_snapshot"]
      f_transfer["dwd.fct_transfer"]
      f_ledger["dwd.fct_ledger_entry"]
      f_funding["dwd.fct_funding_rate"]
      f_pay["dwd.fct_funding_payment"]
      f_ohlcv["dwd.fct_ohlcv_1m"]
      f_bench["dwd.fct_benchmark_nav"]
    end
    subgraph DWS
      d_account["dws.account_day"]
      d_lev["dws.leverage_day"]
      d_fund_nav["dws.fund_nav_day"]
      d_alpha["dws.alpha_day"]
      d_ratio["dws.asset_ratio_day"]
      d_exec["dws.exec_quality_minute"]
      d_risk["dws.risk_events"]
    end
  end

  %% ---------- 应用层 ----------
  subgraph ADS
    vw_port["ads.vw_portfolio_latest"]
    vw_nav["ads.vw_nav_daily"]
    vw_recon["ads.vw_equity_recon_diff"]
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

  %% DWS -> ADS
  d_lev --> vw_port
  d_ratio --> vw_port
  d_alpha --> vw_port
  d_fund_nav --> vw_nav
  d_account --> vw_recon
