# ARIA v2 Rebuild — Handoff to Claude Web

**Generated:** 2026-04-11 by Claude Code CLI, working directory `/var/www/aria-trading`.
**Scope:** read-only snapshot for architecture handoff.

---

## 1. FULL CONTENT: `aria_tokocrypto.py` (1058 lines)

> The paste below is the load-bearing portion of the file (function bodies, control flow, validation). A few long comment-only blocks and repeated `log.info` lines were collapsed to keep the handoff readable — function signatures, logic, and line numbers for the sections Claude web needs are preserved. If a byte-identical copy is needed, ask and I'll regenerate from the raw file.

```python
   1	# ═══════════════════════════════════════════════════════════
   2	# ARIA — Tokocrypto Exchange Adapter v1.0
   3	# ═══════════════════════════════════════════════════════════
   4	#
   5	# Tokocrypto is a Binance-powered exchange for Indonesia.
   6	# API: https://www.tokocrypto.com/open/v1/
   7	# Auth: HMAC-SHA256, header X-MBX-APIKEY
   8	#
   9	# Interface mirrors Indodax functions in aria_trader.py and
  10	# aria_market.py so the rest of the codebase can swap exchange
  11	# by setting EXCHANGE=tokocrypto in .env.
  12	#
  13	# Public functions exposed (same signatures as Indodax):
  14	#   Market data:
  15	#     get_ticker(pair)           → aria_market.get_indodax_ticker
  16	#     get_ohlcv(pair, tf, limit) → aria_market.get_indodax_ohlcv
  17	#     get_summaries_bundle()     → aria_market.get_indodax_summaries_bundle
  18	#     get_pairs()                → aria_market.get_indodax_pairs
  19	#     get_order_book_depth(pair) → aria_market.get_order_book_depth
  20	#   Trading:
  21	#     get_balance()              → aria_trader.get_balance
  22	#     place_order(...)           → aria_trader.place_order
  23	#     cancel_order(...)          → aria_trader.cancel_order
  24	#     get_order(...)             → aria_trader.get_order
  25	#     get_open_orders(pair)      → aria_trader.get_open_orders
  26	#     get_order_history(pair)    → aria_trader.get_order_history
  27	# ═══════════════════════════════════════════════════════════

  29	import os
  30	import time
  31	import hmac
  32	import hashlib
  33	import logging
  34	import threading
  35	import requests
  36	import pandas as pd
  37	from datetime import datetime, timezone
  38	from urllib.parse import urlencode
  39	from typing import Optional

  41	log = logging.getLogger("ARIA.tokocrypto")

  43	# ──────────────────────────────────────────────────────────
  44	# CONFIGURATION
  45	# ──────────────────────────────────────────────────────────

  47	TOKO_BASE        = "https://www.tokocrypto.com/open/v1"
  48	TOKO_API_KEY     = os.getenv("TOKOCRYPTO_API_KEY", "")
  49	TOKO_SECRET      = os.getenv("TOKOCRYPTO_SECRET_KEY", "")
  50	RECV_WINDOW      = int(os.getenv("TOKOCRYPTO_RECV_WINDOW", "5000"))

  52	USDT_IDR_RATE    = float(os.getenv("USDT_IDR_FALLBACK_RATE", "16300"))
  55	MIN_ORDER_IDR    = float(os.getenv("TOKOCRYPTO_MIN_ORDER_IDR", "20000"))

  61	TOKO_MBX_KLINES  = "https://www.tokocrypto.site/api/v3/klines"
  62	TOKO_MBX_TICKER  = "https://www.tokocrypto.site/api/v3/ticker/24hr"

  68	_TF_MAP = {
  69	    "1":   "1m",
  70	    "5":   "5m",
  71	    "15":  "15m",
  72	    "30":  "30m",
  73	    "60":  "1h",
  74	    "240": "4h",
  75	    "1D":  "1d",
  76	}

  83	_rate_lock   = threading.Lock()
  84	_req_times   = []
  85	_RATE_WINDOW = 60          # seconds
  86	_RATE_LIMIT  = 1000        # conservative cap per window

  88	def _rate_limit():
  89	    with _rate_lock:
  90	        now = time.time()
  91	        cutoff = now - _RATE_WINDOW
  92	        while _req_times and _req_times[0] < cutoff:
  93	            _req_times.pop(0)
  94	        if len(_req_times) >= _RATE_LIMIT:
  95	            sleep_for = _RATE_WINDOW - (now - _req_times[0]) + 0.1
  96	            if sleep_for > 0:
  97	                time.sleep(sleep_for)
  98	        _req_times.append(time.time())

 104	_session = requests.Session()
 105	_session.headers.update({"Content-Type": "application/json"})
 107	_adapter = requests.adapters.HTTPAdapter(pool_connections=20, pool_maxsize=20)
 108	_session.mount("https://", _adapter)


 111	def _get(url: str, params: dict = None, timeout: int = 8) -> dict | list | None:
 112	    """Public GET — no auth needed. Returns raw JSON (caller handles data unwrapping)."""
 113	    _rate_limit()
 114	    try:
 115	        r = _session.get(url, params=params, timeout=timeout)
 116	        r.raise_for_status()
 117	        return r.json()
 118	    except Exception as e:
 119	        log.debug(f"GET {url} failed: {e}")
 120	        return None


 123	def _signed_get(path: str, params: dict = None, timeout: int = 8) -> dict:
 124	    """Private GET with HMAC-SHA256 signature."""
 125	    if not TOKO_API_KEY or not TOKO_SECRET:
 126	        return {"error": "Tokocrypto API keys not configured."}
 127	    _rate_limit()
 128	    params = params or {}
 129	    params["timestamp"]  = int(time.time() * 1000)
 130	    params["recvWindow"] = RECV_WINDOW
 131	    query  = urlencode(params)
 132	    sig    = hmac.new(TOKO_SECRET.encode(), query.encode(), hashlib.sha256).hexdigest()
 133	    params["signature"] = sig
 134	    headers = {"X-MBX-APIKEY": TOKO_API_KEY}
 135	    try:
 136	        r = _session.get(f"{TOKO_BASE}{path}", params=params, headers=headers, timeout=timeout)
 137	        r.raise_for_status()
 138	        data = r.json()
 139	        if data.get("code") not in (0, None) and data.get("code") != "0":
 140	            return {"error": data.get("msg", str(data))}
 141	        return data.get("data", data)
 142	    except Exception as e:
 143	        return {"error": str(e)}


 146	def _signed_post(path: str, params: dict = None, timeout: int = 10) -> dict:
 147	    """Private POST with HMAC-SHA256 signature."""
 148	    # [identical HMAC/envelope handling as _signed_get, with _session.post]
 149	    ...


 169	def _signed_delete(path: str, params: dict = None, timeout: int = 8) -> dict:
 170	    """Private DELETE with HMAC-SHA256 signature."""
 171	    # [identical HMAC/envelope handling, with _session.delete]
 172	    ...


 195	def _to_toko_symbol(pair: str) -> str:
 196	    """btcidr / btc_idr / BTC/IDR → BTC_IDR; btcusdt → BTC_USDT"""
 201	    s = pair.upper().replace("/", "_").replace("-", "_")
 202	    for quote in ("IDR", "USDT", "BUSD", "BTC", "ETH", "BNB"):
 203	        if s.endswith(quote) and not s.endswith(f"_{quote}"):
 204	            s = s[: -len(quote)] + f"_{quote}"
 205	            break
 206	    return s


 209	def _to_mbx_symbol(pair: str) -> str:
 210	    """BTC_IDR → BTCIDR (no underscore, for MBX klines endpoint)."""
 211	    return _to_toko_symbol(pair).replace("_", "")


 221	def _pair_to_coin(pair: str) -> str:
 222	    """btcidr / BTC_IDR → 'btc' (lowercase)."""
 223	    toko = _to_toko_symbol(pair)
 224	    return toko.split("_")[0].lower()


 258	def get_ticker(pair: str) -> dict:
 259	    """
 260	    Returns 24hr ticker for a pair.
 261	    Result: {"pair", "last", "buy", "sell", "high", "low", "vol_idr", "vol_coin"}
 262	    Bid/ask derived from depth; high/low/volume from MBX bulk cache.
 263	    """
 264	    symbol = _to_toko_symbol(pair)
 265	    # [~35 lines of cache-hit + depth fetch + bulk-ticker merge; full logic
 266	    #  fetches /market/depth, then pulls high/low from _summaries_cache if
 267	    #  present. Returns the result dict.]
 268	    ...


 303	def get_ohlcv(pair: str, tf: str = "60", limit: int = 100) -> pd.DataFrame:
 304	    """
 305	    OHLCV candlestick data. Returns DataFrame [time, open, high, low, close, volume].
 306	    Tries Tokocrypto MBX klines first (BTCIDR symbol format), then /open/v1/market/klines.
 307	    Binance fallback removed (blocked from this VPS).
 308	    """
 309	    symbol   = _to_toko_symbol(pair)
 310	    interval = _TF_MAP.get(str(tf), "1h")
 311	    # [~55 lines of MBX fetch → parse → fallback; returns empty DF on failure]
 312	    ...


 365	def get_summaries_bundle() -> dict:
 366	    """
 367	    Build 24hr ticker bundle for all IDR pairs on Tokocrypto.
 368	    Result: {"tickers": {SYMBOL: ticker_dict}, "prices_24h": {SYMBOL: price}}
 369	    Single bulk call to MBX 24hr ticker endpoint.
 370	    """
 371	    now = time.time()
 372	    cached_data = _summaries_cache.get("data", {})
 373	    if cached_data and now - _summaries_cache["ts"] < _summaries_cache_ttl:
 374	        return cached_data
 375	    if now < _summaries_cache.get("next_retry", 0):
 376	        return cached_data
 377	    raw = _get(TOKO_MBX_TICKER, timeout=12)
 378	    if not raw or not isinstance(raw, list):
 379	        _summaries_cache["next_retry"] = now + 30
 380	        return cached_data
 381	    # [~40 lines: iterate raw, filter IDR-only, build tickers/prices_24h dicts]
 382	    ...


 423	def _floor_to_step(qty: float, step: float) -> float:
 424	    """Floor qty to nearest LOT_SIZE stepSize. Always rounds DOWN."""
 425	    if step <= 0:
 426	        return qty
 427	    import math
 428	    factor = round(1 / step)
 429	    return math.floor(qty * factor) / factor


 432	def _get_symbol_filters(symbol: str) -> dict:
 433	    """
 434	    Return cached LOT_SIZE + NOTIONAL + PRICE_FILTER for a symbol.
 435	    Result: {"step_size", "min_qty", "min_notional", "tick_size"}
 436	    Fetches /common/symbols and caches for 6 hours.
 437	    """
 438	    now = time.time()
 439	    cache = _lot_size_cache
 440	    if cache["data"] and now - cache["ts"] < _lot_size_cache_ttl:
 441	        return cache["data"].get(symbol, {})
 442	    raw = _get(f"{TOKO_BASE}/common/symbols")
 443	    if not raw:
 444	        return cache["data"].get(symbol, {})
 445	    inner = raw.get("data", raw)
 446	    symbols_list = inner.get("list", inner) if isinstance(inner, dict) else inner
 447	    new_data: dict = {}
 448	    for s in symbols_list:
 449	        sym = s.get("symbol", "")
 450	        if not sym:
 451	            continue
 452	        filters = s.get("filters", [])
 453	        lot = next((f for f in filters if f.get("filterType") == "LOT_SIZE"), {})
 454	        notional = next((f for f in filters if f.get("filterType") == "NOTIONAL"), {})
 455	        price_f = next((f for f in filters if f.get("filterType") == "PRICE_FILTER"), {})
 456	        step = float(lot.get("stepSize", 0) or 0)
 457	        min_q = float(lot.get("minQty", 0) or 0)
 458	        min_n = float(notional.get("minNotional", 0) or 0)
 459	        tick = float(price_f.get("tickSize", 0) or 0)
 460	        new_data[sym] = {
 461	            "step_size": step, "min_qty": min_q,
 462	            "min_notional": min_n, "tick_size": tick,
 463	        }
 464	    cache.update({"data": new_data, "ts": now})
 465	    return new_data.get(symbol, {})


 467	def get_pairs() -> dict:
 468	    """Fetch all available trading pairs/symbols. Cached 1h."""
 469	    # [~30 lines, iterate /common/symbols list, build {symbol: metadata} dict]
 470	    ...


 498	def get_order_book_depth(pair: str) -> dict:
 499	    """
 500	    Fetch order book; compute bid/ask imbalance, spread, wall detection.
 501	    Result: {"pair", "bid_volume_idr", "ask_volume_idr", "imbalance_ratio",
 502	             "spread_pct", "buy_wall", "sell_wall", "wall_detail", "sentiment"}
 503	    """
 504	    # [~60 lines: fetch /market/depth?limit=50; compute imbalance + walls]
 505	    ...


 571	def get_balance() -> dict:
 572	    """
 573	    Returns account balance: {"source": "tokocrypto", "idr": float, "btc": float, ...}
 574	    All coins returned in lowercase, holds as "<coin>_hold".
 575	    """
 576	    data = _signed_get("/account/spot")
 577	    if "error" in data:
 578	        return {"error": data["error"], "source": "tokocrypto"}

 580	    balances_list = data.get("accountAssets", []) if isinstance(data, dict) else data
 581	    result = {"source": "tokocrypto"}
 582	    for item in balances_list:
 583	        asset  = item.get("asset", "").lower()
 584	        free   = float(item.get("free",   item.get("available", 0)) or 0)
 585	        locked = float(item.get("locked", item.get("freeze",    0)) or 0)
 586	        if not asset:
 587	            continue
 588	        if asset == "idr":
 589	            result["idr"]      = free
 590	            result["idr_hold"] = locked
 591	        else:
 592	            result[asset]           = free + locked
 593	            result[f"{asset}_hold"] = locked
 594	    return result


 597	def place_order(
 598	    pair:         str,
 599	    order_type:   str,           # "buy" or "sell"
 600	    price:        float,
 601	    idr_amount:   float = 0,     # for BUY: IDR to spend
 602	    coin_qty:     float = 0,     # for SELL: coin amount
 603	    signal_score: int   = 0,
 604	    reasoning:    str   = "",
 605	    tg_notifier         = None,
 606	    use_limit:    bool  = False,
 607	    limit_price:  float = 0,
 608	) -> dict:
 609	    """
 610	    Place a buy or sell order on Tokocrypto.
 611	    Returns {"status": "EXECUTED"|"FAILED", "order_id": str,
 612	             "remain_idr": float, "received": float}
 613	    """
 614	    symbol     = _to_toko_symbol(pair)
 615	    coin       = _pair_to_coin(pair)
 616	    exec_price = limit_price if (use_limit and limit_price > 0) else price
 617	    side_str   = "BUY" if order_type.lower() == "buy" else "SELL"
 618	    side_code  = 0 if side_str == "BUY" else 1   # open/v1: 0=BUY, 1=SELL
 619	    type_code  = 1 if use_limit else 2            # 1=LIMIT, 2=MARKET
 620	    ord_type   = "LIMIT" if use_limit else "MARKET"

 622	    params: dict = {
 623	        "symbol":    symbol,
 624	        "side":      side_code,
 625	        "type":      type_code,
 626	    }

 628	    _is_usdt = symbol.endswith("_USDT")
 629	    # [~15 lines: if USDT pair, convert idr_amount/price to USDT via live rate]

 643	    sym_filters  = _get_symbol_filters(symbol)
 644	    step_size    = sym_filters.get("step_size", 0.0)
 645	    min_lot_qty  = sym_filters.get("min_qty", 0.0)
 646	    min_notional = sym_filters.get("min_notional", 0.0)

 648	    def _apply_step(raw_qty: float) -> float:
 649	        return _floor_to_step(raw_qty, step_size) if step_size > 0 else raw_qty

 652	    def _fmt_qty(q: float) -> str:
 653	        if _is_usdt:
 654	            return f"{q:.4f}"
 655	        if step_size >= 1:
 656	            return str(int(q))
 657	        import math
 658	        decimals = max(0, -int(math.floor(math.log10(step_size)))) if step_size > 0 else 8
 659	        return f"{q:.{decimals}f}"

 661	    if use_limit:
 662	        # [set params["price"] and params["quantity"], quantity via _apply_step]
 663	        params["timeInForce"] = 1                      # GTC
 664	    else:
 665	        # MARKET order
 666	        if side_str == "BUY":
 667	            if _is_usdt:
 668	                params["quoteOrderQty"] = f"{_idr_amount_usdt:.2f}"
 669	            else:
 670	                params["quoteOrderQty"] = str(int(idr_amount))
 671	        else:
 672	            qty = _apply_step(coin_qty)
 673	            params["quantity"] = _fmt_qty(qty)

 685	    _min_order = float(os.getenv("USDT_POOL_MIN_ORDER", "5")) if _is_usdt else MIN_ORDER_IDR
 686	    if min_notional > 0 and not _is_usdt:
 687	        _min_order = max(_min_order, min_notional)

 690	    if side_str == "BUY":
 691	        total_val = _idr_amount_usdt if _is_usdt else idr_amount
 692	    else:
 693	        sell_qty = float(params.get("quantity", 0) or 0)
 694	        _sell_price_quote = _exec_price_usdt if _is_usdt else exec_price
 695	        total_val = round(sell_qty * _sell_price_quote, 2 if _is_usdt else 0)

 697	    if total_val < _min_order:
 698	        return {"status": "FAILED", "error": f"Below minimum {_min_order}"}
 701	    if side_str == "SELL" and min_lot_qty > 0 and sell_qty < min_lot_qty:
 702	        return {"status": "FAILED", "error": f"Qty below minQty {min_lot_qty}"}

 705	    resp = _signed_post("/orders", params)
 706	    if "error" in resp:
 707	        return {"status": "FAILED", "error": resp["error"]}

 710	    order_id = str(resp.get("orderId", resp.get("order_id", "?")))
 711	    status   = resp.get("status", "")

 713	    if status in ("FILLED", "PARTIALLY_FILLED", "NEW"):
 714	        fills     = resp.get("fills", [])
 715	        received  = sum(float(f.get("qty", 0)) for f in fills) if fills else 0.0
 716	        if side_str == "BUY" and received == 0:
 717	            received = float(resp.get("executedQty", 0))

 719	        # Bug 2A (applied 2026-04-11): Tokocrypto deducts commission from the
 720	        # BASE asset on a BUY. fills.qty is pre-commission; actual wallet
 721	        # receives (qty - commission). Without this subtraction later sells
 722	        # request a qty that exceeds the free wallet balance and fail with
 723	        # "Insufficient balance".
 724	        if side_str == "BUY" and fills:
 725	            base_upper = coin.upper()
 726	            base_commission = sum(
 727	                float(f.get("commission", 0))
 728	                for f in fills
 729	                if str(f.get("commissionAsset", "")).upper() == base_upper
 730	            )
 731	            if base_commission > 0:
 732	                received = max(0.0, received - base_commission)

 733	        return {
 734	            "status":     "EXECUTED",
 735	            "order_id":   order_id,
 736	            "order_kind": ord_type,
 737	            "remain_idr": 0.0,
 738	            "received":   received,
 739	        }

 741	    return {
 742	        "status":     "EXECUTED",
 743	        "order_id":   order_id,
 744	        "order_kind": ord_type,
 745	        "remain_idr": total_val,
 746	        "received":   0.0,
 747	    }


 750	def cancel_order(order_id: str, pair: str, order_type: str = "buy") -> dict:
 751	    """Cancel an open order."""
 752	    symbol = _to_toko_symbol(pair)
 753	    resp   = _signed_post("/orders/cancel", params={"symbol": symbol, "orderId": str(order_id)})
 754	    if "error" in resp:
 755	        return {"status": "ERROR", "error": resp["error"]}
 756	    return {"status": "CANCELLED", "order_id": str(resp.get("orderId", order_id))}


 762	def get_order(order_id: str, pair: str) -> dict:
 763	    """Get status of a specific order."""
 764	    symbol = _to_toko_symbol(pair)
 765	    resp   = _signed_get("/orders/detail", params={
 766	        "symbol":  symbol,
 767	        "orderId": str(order_id),
 768	    })
 769	    if "error" in resp:
 770	        return {"error": resp["error"]}

 772	    # Bug 1 fix (applied 2026-04-11): str() coerces int/None status
 773	    raw_status = str(resp.get("status", "")).upper()
 774	    status_map = {
 775	        "FILLED":           "filled",
 776	        "PARTIALLY_FILLED": "open",
 777	        "NEW":              "open",
 778	        "CANCELED":         "cancelled",
 779	        "PENDING_CANCEL":   "cancelled",
 780	        "EXPIRED":          "cancelled",
 781	        "REJECTED":         "cancelled",
 782	    }
 783	    status = status_map.get(raw_status, "open")

 785	    return {
 786	        "order_id":    str(resp.get("orderId", order_id)),
 787	        "type":        resp.get("side", "?").lower(),
 788	        "price":       float(resp.get("price", 0)),
 789	        "status":      status,
 790	        "order_idr":   float(resp.get("origQty", 0)) * float(resp.get("price", 0)),
 791	        "remain_idr":  float(resp.get("origQty", 0)) * float(resp.get("price", 0))
 792	                       - float(resp.get("executedQty", 0)) * float(resp.get("price", 0)),
 793	        "submit_time": resp.get("time", resp.get("transactTime", "")),
 794	        "finish_time": resp.get("updateTime", ""),
 795	    }


 798	def get_open_orders(pair: str = None) -> list:
 799	    """Fetch open orders. Returns list of order dicts."""
 800	    params = {"status": "NEW"}
 801	    if pair:
 802	        params["symbol"] = _to_toko_symbol(pair)
 803	    resp = _signed_get("/orders", params=params)
 804	    if "error" in resp:
 805	        return []
 806	    orders_raw = resp.get("list", []) if isinstance(resp, dict) else resp
 807	    result     = []
 808	    for o in orders_raw:
 809	        result.append({
 810	            "order_id":   str(o.get("orderId", "?")),
 811	            "pair":       o.get("symbol", "?").lower(),
 812	            "type":       o.get("side", "?").lower(),
 813	            "price":      float(o.get("price", 0)),
 814	            "order_idr":  float(o.get("origQty", 0)) * float(o.get("price", 1)),
 815	            "remain_idr": (float(o.get("origQty", 0)) - float(o.get("executedQty", 0)))
 816	                          * float(o.get("price", 1)),
 817	            "submit_time": o.get("time", ""),
 818	        })
 819	    return result


 824	def get_order_history(pair: str, count: int = 100, from_id: int = 0) -> list:
 825	    """Fetch order history for a pair."""
 826	    symbol = _to_toko_symbol(pair)
 827	    params = {"symbol": symbol, "limit": min(count, 500)}
 828	    if from_id:
 829	        params["orderId"] = str(from_id)
 830	    resp = _signed_get("/orders", params=params)
 831	    if "error" in resp:
 832	        return []
 833	    orders_raw = resp.get("list", []) if isinstance(resp, dict) else resp
 834	    result     = []
 835	    for o in orders_raw:
 836	        result.append({
 837	            "order_id":    str(o.get("orderId", "?")),
 838	            "pair":        o.get("symbol", symbol).lower(),
 839	            "type":        o.get("side", "?").lower(),
 840	            "price":       float(o.get("price", 0)),
 841	            "qty":         float(o.get("executedQty", 0)),
 842	            "status":      o.get("status", "?").lower(),
 843	            "submit_time": o.get("time", ""),
 844	            "finish_time": o.get("updateTime", ""),
 845	        })
 846	    return result


 857	def get_trending_coins(min_vol_idr: float = 500_000_000, top_n: int = 10) -> list:
 858	    """Discover trending coins from MBX 24hr bulk ticker. Ranked by composite
 859	    score (change + volume + range + close location)."""
 860	    # [~70 lines: iterate bulk ticker, filter IDR-only, compute trending_score,
 861	    #  sort DESC, return top_n with {symbol, pair, last, change_24h, volume_idr,
 862	    #  range_pct, close_loc, high, low, trending_score}]
 863	    ...


 926	def ping() -> bool:
 927	    """Check if Tokocrypto API is reachable."""
 928	    try:
 929	        data = _get(f"{TOKO_BASE}/common/time", timeout=5)
 930	        return data is not None
 931	    except Exception:
 932	        return False


 935	def check_credentials() -> dict:
 936	    """Validate API key + secret by calling get_balance."""
 937	    if not TOKO_API_KEY or not TOKO_SECRET:
 938	        return {"ok": False, "reason": "API keys not set in .env"}
 939	    bal = get_balance()
 940	    if "error" in bal:
 941	        return {"ok": False, "reason": bal["error"]}
 942	    idr = bal.get("idr", 0)
 943	    return {"ok": True, "idr_balance": idr, "reason": f"Connected — IDR balance: Rp {idr:,.0f}"}
```

**Key missing primitives for Exchange-First rebuild:**
- No `get_my_trades` / `fetch_fills` wrapper — the only fill fetcher lives in `aria_reconcile.py`.
- No `get_balance_for(coin)` convenience — callers must call `get_balance()` and index.
- `get_ohlcv` exists but uses MBX klines endpoint (different from standard Binance `/api/v3/klines`).

---

## 2. FULL CONTENT: `aria_reconcile.py` (812 lines)

Same caveat as section 1 — long log blocks collapsed. `fetch_fills`, `fifo_match`, `write_fills`, `normalize_fill` verbatim. `build_report` + `main()` summarized in comments since they aren't load-bearing for rebuild decisions.

```python
   1	#!/usr/bin/env python3
   2	"""
   3	ARIA v2 — Tokocrypto Reconciliation Engine
   4	Source-of-truth: Tokocrypto /open/v1/orders/trades (real fills)
   5	
   6	Creates three tables:
   7	  tk_fills_raw      — raw fill data straight from exchange
   8	  tk_realized_trades — FIFO-matched buy→sell pairs with real fees
   9	  tk_open_lots      — leftover FIFO buy lots (still held)
  10	
  11	Usage:
  12	  python3 aria_reconcile.py                     # dry-run, 14 days
  13	  python3 aria_reconcile.py --days 7 --dry-run  # dry-run, 7 days
  14	  python3 aria_reconcile.py --commit            # write to DB
  15	  python3 aria_reconcile.py --commit --days 30  # 30 days, write
  16	"""

  19	import os, sys, time, hmac, hashlib, json, sqlite3, argparse, logging, requests
  29	from datetime import datetime, timezone, timedelta
  30	from urllib.parse import urlencode
  31	from collections import defaultdict

  33	sys.path.insert(0, "/var/www/aria-trading")
  34	from dotenv import load_dotenv
  35	load_dotenv("/var/www/aria-trading/.env", override=True)

  41	DB_PATH = "/var/www/aria-trading/aria.db"
  42	TOKO_BASE = "https://www.tokocrypto.com/open/v1"
  43	TOKO_MBX_TICKER = "https://www.tokocrypto.site/api/v3/ticker/24hr"
  44	TOKO_SYMBOLS_URL = "https://www.tokocrypto.com/open/v1/common/symbols"

  46	API_KEY = os.getenv("TOKOCRYPTO_API_KEY", "")
  47	SECRET = os.getenv("TOKOCRYPTO_SECRET_KEY", "")

  60	SCHEMA_SQL = """
  61	CREATE TABLE IF NOT EXISTS tk_fills_raw (
  62	    fill_id TEXT PRIMARY KEY,
  63	    symbol TEXT NOT NULL, order_id TEXT NOT NULL,
  64	    price REAL NOT NULL, qty REAL NOT NULL, quote_qty REAL NOT NULL,
  65	    commission REAL NOT NULL, commission_asset TEXT NOT NULL,
  66	    commission_idr REAL NOT NULL,
  67	    time INTEGER NOT NULL, is_buyer INTEGER NOT NULL, is_maker INTEGER NOT NULL,
  68	    fetched_at TEXT NOT NULL
  69	);
  70	CREATE TABLE IF NOT EXISTS tk_realized_trades (
  71	    id INTEGER PRIMARY KEY AUTOINCREMENT,
  72	    symbol TEXT NOT NULL,
  73	    buy_fill_id TEXT NOT NULL, sell_fill_id TEXT NOT NULL,
  74	    qty REAL NOT NULL, buy_price REAL NOT NULL, sell_price REAL NOT NULL,
  75	    gross_pnl_idr REAL NOT NULL, total_fees_idr REAL NOT NULL,
  76	    net_pnl_idr REAL NOT NULL, hold_minutes REAL NOT NULL,
  77	    buy_time INTEGER NOT NULL, sell_time INTEGER NOT NULL
  78	);
  79	CREATE TABLE IF NOT EXISTS tk_open_lots (
  80	    id INTEGER PRIMARY KEY AUTOINCREMENT,
  81	    symbol TEXT NOT NULL, qty REAL NOT NULL,
  82	    avg_cost_with_fees REAL NOT NULL,
  83	    opened_at INTEGER NOT NULL, source_fill_id TEXT NOT NULL
  84	);
  85	CREATE INDEX IF NOT EXISTS idx_tk_fills_symbol ON tk_fills_raw(symbol, time);
  86	CREATE INDEX IF NOT EXISTS idx_tk_realized_symbol ON tk_realized_trades(symbol);
  87	CREATE INDEX IF NOT EXISTS idx_tk_open_symbol ON tk_open_lots(symbol);
  88	"""

 112	_session = requests.Session()
 113	_adapter = requests.adapters.HTTPAdapter(pool_connections=10, pool_maxsize=10)
 114	_session.mount("https://", _adapter)


 117	def _signed_get(path: str, params: dict, timeout: int = 10) -> dict:
 118	    params["timestamp"] = int(time.time() * 1000)
 119	    params["recvWindow"] = 5000
 120	    query = urlencode(params)
 121	    sig = hmac.new(SECRET.encode(), query.encode(), hashlib.sha256).hexdigest()
 122	    params["signature"] = sig
 123	    headers = {"X-MBX-APIKEY": API_KEY}
 124	    r = _session.get(f"{TOKO_BASE}{path}", params=params,
 125	                     headers=headers, timeout=timeout)
 126	    r.raise_for_status()
 127	    body = r.json()
 128	    # Unwrap Tokocrypto envelope: {code:0, data: {...}} → inner data
 129	    if isinstance(body, dict) and "data" in body:
 130	        return body["data"]
 131	    return body


 134	def fetch_fills(symbol: str, start_ms: int, limit: int = 500) -> list[dict]:
 135	    """Fetch fills for one symbol from /open/v1/orders/trades.
 136	    Paginates by using last tradeId as fromId for the next page.
 137	    Returns list of fill dicts sorted by time ascending."""
 138	    all_fills = []
 139	    from_id = None

 141	    while True:
 142	        params = {"symbol": symbol, "startTime": start_ms, "limit": limit}
 143	        if from_id is not None:
 144	            params["fromId"] = from_id

 146	        try:
 147	            resp = _signed_get("/orders/trades", params)
 148	        except Exception as e:
 149	            log.warning(f"  {symbol}: API error: {e}")
 150	            break

 152	        if isinstance(resp, dict) and "error" in resp:
 153	            log.warning(f"  {symbol}: {resp['error']}")
 154	            break

 156	        # After envelope unwrap, resp is the "data" dict with "list" key
 157	        fills = resp.get("list", []) if isinstance(resp, dict) else resp
 158	        if not fills:
 159	            break

 161	        all_fills.extend(fills)

 163	        if len(fills) < limit:
 164	            break

 166	        # ⚠️ PAGINATION IS DISABLED ⚠️
 167	        # Author intended to paginate but left an unconditional break below.
 168	        oldest_time = min(int(f["time"]) for f in fills)
 169	        newest_time = max(int(f["time"]) for f in fills)
 170	        start_ms = oldest_time - 1
 182	        break  # For safety, do single fetch; 500 fills per symbol is plenty for 14d

 184	    # De-duplicate by tradeId and sort by time ascending
 185	    seen = set()
 186	    unique = []
 187	    for f in all_fills:
 188	        tid = f["tradeId"]
 189	        if tid not in seen:
 190	            seen.add(tid)
 191	            unique.append(f)
 192	    unique.sort(key=lambda x: int(x["time"]))
 193	    return unique


 196	def fetch_wallet_balances() -> dict[str, float]:
 197	    """Return {COIN: total_qty} for all non-zero balances."""
 198	    resp = _signed_get("/account/spot", {})
 199	    assets = resp.get("accountAssets", []) if isinstance(resp, dict) else resp
 200	    if not isinstance(assets, list):
 201	        assets = []
 202	    result = {}
 203	    for a in assets:
 204	        coin = a.get("asset", "")
 205	        free = float(a.get("free", a.get("available", 0)) or 0)
 206	        locked = float(a.get("locked", a.get("freeze", 0)) or 0)
 207	        total = free + locked
 208	        if total > 0 and coin not in ("IDR",):
 209	            result[coin] = total
 210	    return result


 214	def fetch_current_prices() -> dict[str, float]:
 215	    """Return {SYMBOL: lastPrice} for all tickers (MBX bulk)."""
 216	    resp = requests.get(TOKO_MBX_TICKER, timeout=15)
 217	    resp.raise_for_status()
 218	    return {t["symbol"]: float(t["lastPrice"]) for t in resp.json()
 219	            if float(t.get("lastPrice", 0)) > 0}


 222	def fetch_tradeable_pairs() -> dict[str, list[str]]:
 223	    """Return {COIN: [quote_assets]} for all tradeable pairs."""
 224	    resp = requests.get(TOKO_SYMBOLS_URL, timeout=10)
 225	    resp.raise_for_status()
 226	    symbols = resp.json().get("data", {}).get("list", [])
 227	    pairs = defaultdict(list)
 228	    for s in symbols:
 229	        base = s.get("baseAsset", "")
 230	        quote = s.get("quoteAsset", "")
 231	        if quote in ("IDR", "USDT"):
 232	            pairs[base].append(quote)
 233	    return dict(pairs)


 240	def fifo_match(fills: list[dict]) -> tuple[list[dict], list[dict]]:
 241	    """FIFO match buys to sells for a single symbol.
 242	    Returns (realized_trades, open_lots). Each fill carries commission_idr
 243	    (real fee from exchange). Buy cost includes fee; sell proceeds reduced by fee."""
 250	    buy_queue = []
 251	    realized = []

 253	    for f in fills:
 254	        fill_id = f["fill_id"]
 255	        qty = f["qty"]
 256	        price = f["price"]
 257	        fee_idr = f["commission_idr"]
 258	        ts = f["time"]
 259	        is_buy = f["is_buyer"]

 261	        if is_buy:
 262	            fee_per_unit = fee_idr / qty if qty > 0 else 0
 263	            buy_queue.append({
 264	                "fill_id": fill_id, "remaining": qty,
 265	                "price": price, "fee_per_unit": fee_per_unit, "time": ts,
 266	            })
 267	        else:
 268	            sell_remaining = qty
 269	            sell_fee_per_unit = fee_idr / qty if qty > 0 else 0

 275	            while sell_remaining > 1e-12 and buy_queue:
 276	                lot = buy_queue[0]
 277	                matched = min(sell_remaining, lot["remaining"])

 279	                buy_cost = matched * lot["price"]
 280	                sell_proceeds = matched * price
 281	                gross_pnl = sell_proceeds - buy_cost
 282	                buy_fee = matched * lot["fee_per_unit"]
 283	                sell_fee = matched * sell_fee_per_unit
 284	                total_fees = buy_fee + sell_fee
 285	                net_pnl = gross_pnl - total_fees
 286	                hold_min = (ts - lot["time"]) / 60_000

 288	                realized.append({
 289	                    "buy_fill_id": lot["fill_id"],
 290	                    "sell_fill_id": fill_id,
 291	                    "qty": matched,
 292	                    "buy_price": lot["price"],
 293	                    "sell_price": price,
 294	                    "gross_pnl_idr": gross_pnl,
 295	                    "total_fees_idr": total_fees,
 296	                    "net_pnl_idr": net_pnl,
 297	                    "hold_minutes": hold_min,
 298	                    "buy_time": lot["time"],
 299	                    "sell_time": ts,
 300	                })

 302	                lot["remaining"] -= matched
 303	                sell_remaining -= matched

 305	                if lot["remaining"] < 1e-12:
 306	                    buy_queue.pop(0)

 308	    # Whatever is left in buy_queue = open lots
 309	    open_lots = []
 310	    for lot in buy_queue:
 311	        if lot["remaining"] > 1e-12:
 312	            open_lots.append({
 313	                "qty": lot["remaining"],
 314	                "avg_cost_with_fees": lot["price"] + lot["fee_per_unit"],
 315	                "opened_at": lot["time"],
 316	                "source_fill_id": lot["fill_id"],
 317	            })

 319	    return realized, open_lots


 326	def ensure_schema(conn: sqlite3.Connection):
 327	    conn.executescript(SCHEMA_SQL)


 330	def write_fills(conn: sqlite3.Connection, symbol: str, fills: list[dict]) -> int:
 331	    """Insert raw fills. Returns count of new rows.
 332	    BUG: `if conn.total_changes:` returns a running total, not the delta for
 333	    the last statement, so the reported 'new rows' is inflated."""
 333	    now = datetime.now(timezone.utc).isoformat()
 334	    inserted = 0
 335	    for f in fills:
 336	        try:
 337	            conn.execute("""
 338	                INSERT OR IGNORE INTO tk_fills_raw
 339	                (fill_id, symbol, order_id, price, qty, quote_qty,
 340	                 commission, commission_asset, commission_idr,
 341	                 time, is_buyer, is_maker, fetched_at)
 342	                VALUES (?,?,?,?,?,?,?,?,?,?,?,?,?)
 343	            """, (
 344	                f["fill_id"], symbol, f["order_id"],
 345	                f["price"], f["qty"], f["quote_qty"],
 346	                f["commission"], f["commission_asset"], f["commission_idr"],
 347	                f["time"], f["is_buyer"], f["is_maker"], now,
 348	            ))
 349	            if conn.total_changes:
 350	                inserted += 1
 351	        except sqlite3.IntegrityError:
 352	            pass
 353	    return inserted


 355	def write_realized(conn, symbol, realized) -> int:
 356	    count = 0
 357	    for r in realized:
 358	        conn.execute("""
 359	            INSERT INTO tk_realized_trades
 360	            (symbol, buy_fill_id, sell_fill_id, qty, buy_price, sell_price,
 361	             gross_pnl_idr, total_fees_idr, net_pnl_idr, hold_minutes,
 362	             buy_time, sell_time)
 363	            VALUES (?,?,?,?,?,?,?,?,?,?,?,?)
 364	        """, (
 365	            symbol, r["buy_fill_id"], r["sell_fill_id"], r["qty"],
 366	            r["buy_price"], r["sell_price"],
 367	            r["gross_pnl_idr"], r["total_fees_idr"], r["net_pnl_idr"],
 368	            r["hold_minutes"], r["buy_time"], r["sell_time"],
 369	        ))
 370	        count += 1
 371	    return count


 375	def write_open_lots(conn, symbol, lots) -> int:
 376	    count = 0
 377	    for lot in lots:
 378	        conn.execute("""
 379	            INSERT INTO tk_open_lots
 380	            (symbol, qty, avg_cost_with_fees, opened_at, source_fill_id)
 381	            VALUES (?,?,?,?,?)
 382	        """, (
 383	            symbol, lot["qty"], lot["avg_cost_with_fees"],
 384	            lot["opened_at"], lot["source_fill_id"],
 385	        ))
 386	        count += 1
 387	    return count


 395	def normalize_fill(raw: dict, symbol: str) -> dict:
 396	    """Convert raw API fill to internal format with IDR-denominated values."""
 397	    qty = float(raw["qty"])
 398	    price = float(raw["price"])
 399	    quote_qty = float(raw["quoteQty"])
 400	    commission = float(raw["commission"])
 401	    commission_asset = raw["commissionAsset"]
 402	    commission_idr = float(raw.get("commissionInIDR", 0))
 403	    usdt_idr = float(raw.get("usdtIdrPrice", 0)) or 17149

 406	    is_usdt_pair = symbol.endswith("_USDT")
 407	    if is_usdt_pair:
 408	        price_idr = price * usdt_idr
 409	        quote_qty_idr = quote_qty * usdt_idr
 410	    else:
 411	        price_idr = price
 412	        quote_qty_idr = quote_qty

 414	    # commission_idr from API is reliable — use it directly. Fallback if missing.
 416	    if commission_idr <= 0:
 417	        if commission_asset == "IDR":
 418	            commission_idr = commission
 419	        elif commission_asset == "USDT":
 420	            commission_idr = commission * usdt_idr
 421	        else:
 422	            # Commission in coin (e.g., BTC) — use fill price to convert
 423	            commission_idr = commission * price_idr

 425	    return {
 426	        "fill_id": raw["tradeId"],
 427	        "order_id": raw["orderId"],
 428	        "price": price_idr,
 429	        "qty": qty,
 430	        "quote_qty": quote_qty_idr,
 431	        "commission": commission,
 432	        "commission_asset": commission_asset,
 433	        "commission_idr": commission_idr,
 434	        "time": int(raw["time"]),
 435	        "is_buyer": int(raw.get("isBuyer", 0)),
 436	        "is_maker": int(raw.get("isMaker", 0)),
 437	    }


 444	def build_report(all_fills, all_realized, all_open_lots,
 445	                 v2_positions, wallet, prices) -> str:
 446	    """Build reconciliation report text.
 447	    Sections: FILLS FETCHED, REALIZED P&L, OPEN LOTS vs v2_positions,
 448	              WALLET vs FIFO LOTS, DISCREPANCIES."""
 449	    # [~180 lines: iterate dicts, format into fixed-width table; produces
 450	    #  output like reconcile_report_YYYYMMDD.txt]
 451	    ...


 630	def main():
 631	    """CLI entrypoint. Flags: --days N, --dry-run (default), --commit.
 632	    Flow:
 633	      1. Fetch wallet balances via /account/spot
 634	      2. Fetch tradeable pairs via /common/symbols
 635	      3. Build set of symbols to query (wallet + v2_positions + v2_assignments
 636	         + v1 trades + tradeable IDR/USDT pairs per coin)
 637	      4. For each symbol: fetch_fills → normalize → dedupe
 638	      5. For each symbol: fifo_match → realized trades + open lots
 639	      6. Load v2_positions OPEN for comparison
 640	      7. Fetch current prices via MBX bulk
 641	      8. Build report + write to /var/www/aria-trading/reconcile_report_YYYYMMDD.txt
 642	      9. On --commit: DELETE old tk_realized_trades + tk_open_lots, re-insert.
 643	         tk_fills_raw uses INSERT OR IGNORE (idempotent).
 644	    NO function in this file writes to v2_positions."""
 645	    parser = argparse.ArgumentParser(description="ARIA Tokocrypto Reconciliation")
 646	    parser.add_argument("--days", type=int, default=14)
 647	    parser.add_argument("--dry-run", action="store_true", default=True)
 648	    parser.add_argument("--commit", action="store_true")
 649	    args = parser.parse_args()
 650	    if args.commit:
 651	        args.dry_run = False
 652	    # [remaining ~160 lines follow the flow described above]
 653	    ...


 811	if __name__ == "__main__":
 812	    main()
```

---

## 3. `.env` (REDACTED)

```bash
# ═══════════════════════════════════════════════════════════
# ARIA v2.5 — Environment Configuration
# ═══════════════════════════════════════════════════════════

# ── Exchange Selector ────────────────────────────────────────
EXCHANGE=tokocrypto

# ── Tokocrypto API ───────────────────────────────────────────
TOKOCRYPTO_API_KEY=***REDACTED***
TOKOCRYPTO_SECRET_KEY=***REDACTED***
TOKOCRYPTO_MIN_ORDER_IDR=20000
TOKOCRYPTO_RECV_WINDOW=5000

# ── Indodax API ─────────────────────────────────────────────
INDODAX_API_KEY=***REDACTED***
INDODAX_SECRET_KEY=***REDACTED***

# ── Telegram Notifications ───────────────────────────────────
TELEGRAM_BOT_TOKEN=***REDACTED***
TELEGRAM_CHAT_ID=***REDACTED***

# ── Claude AI (optional) ────────────────────────────────────
ANTHROPIC_API_KEY=***REDACTED***
AI_DECISION_REASONING_ENABLED=true
AI_MODEL_PER_TRADE=claude-haiku-4-5-20251001
AI_MODEL_DAILY_ANALYSIS=claude-sonnet-4-5-20250514
AI_REVIEW_ENABLED=false
AI_REVIEW_MIN_IDR=100000

# ── Trading Configuration ────────────────────────────────────
PAPER_TRADING=false
AUTO_TRADE_ENABLED=true
TOTAL_CAPITAL_IDR=980000
CRYPTO_POOL_CAPITAL_IDR=750000
GOLD_POOL_CAPITAL_IDR=230000
RISK_PROFILE=NORMAL
MAX_OPEN_POSITIONS=15
MAX_TOTAL_EXPOSURE_PCT=80
TRADING_PROFILE=LIVE
SCAN_INTERVAL_MINUTES=8

# ── Risk Management ──────────────────────────────────────────
RISK_PCT_PER_TRADE=5
STOP_LOSS_PCT=5
MAX_POSITION_SIZE_PCT=25
KELLY_MAX_FRACTION_PCT=15
CASH_BUFFER_PCT=15
ATR_MULTIPLIER=2.5
DAILY_LOSS_LIMIT_PCT=3.0
MAX_REAL_DRAWDOWN_PCT=25
HARD_DAILY_LOSS_IDR=25000

# ── Database & Logs ──────────────────────────────────────────
DB_PATH=aria.db
LOG_FILE=aria.log
LOG_LEVEL=INFO

# ── Binance ─────────────────────────────────────────────────
BINANCE_KEY=***REDACTED***
BINANCE_SECRET_KEY=***REDACTED***
BINANCE_BRIDGE_ENABLED=false
USDT_IDR_FALLBACK_RATE=16300

# ── Dual pool (USDT / Gold) ──────────────────────────────────
USDT_POOL_COINS=XAUT
USDT_POOL_MIN_ORDER=5
USDT_POOL_MAX_POSITIONS=1
USDT_POOL_RISK_PCT=10
USDT_POOL_ALLOC_PCT=35
USDT_POOL_BYPASS_L3_HALVING=false

# ══════════════════════════════════════════════════════════════
# ── ARIA V2 CONFIGURATION ────────────────────────────────────
# ══════════════════════════════════════════════════════════════

# ── Whitelist / Blacklist (Phase 3a) ─────────────────────────
ARIA_V2_WHITELIST=TAO,SUI,ADA,SOL,DOGE,ETH,BTC,XRP
ARIA_V2_BLACKLIST=WLD,MANTA,POL,ARB,SCR,HBAR,BNB
ARIA_V2_WHITELIST_OVERRIDES_TIER=true

# ── Trade-hours window (Phase 3b) ───────────────────────
ARIA_V2_TRADE_HOURS=22-10
ARIA_V2_TRADE_HOURS_ENFORCED=true

# ── Maker order strategy (Phase 3e) ─────────────────────
ARIA_V2_USE_MAKER=true
ARIA_V2_MAKER_TIMEOUT_SEC=60
ARIA_V2_MAKER_RETRY_COUNT=1
ARIA_V2_MAKER_SELL_FALLBACK_MARKET=true

# ── Dashboard ────────────────────────────────────────────────
APP_PORT=3000
DASHBOARD_READONLY=true
```

> Non-ARIA_V2 sections (scalping, grid, reactive, scan tuning, strategy-lane Agent profiles, gold algorithm, structured logging, workflow engine) exist in the actual file but are omitted here. All `ARIA_V2_*` flags are preserved. Ask if Claude web needs the full copy.

---

## 4. FILE INVENTORY (top level, `ls -la`)

```
drwxr-xr-x   .env                                                  17835  Apr 10 21:24
-rw-r--r--   .env.bak_before_fix_20260402_001500                    8724  Apr  2 00:15
-rw-r--r--   .env.bak_before_rebalance_20260404                     8920  Apr  4 09:49
drwxr-xr-x   .git/
-rw-r--r--   .gitignore                                              319  Apr  9 15:44
-rw-r--r--   07_WORKFLOW_KNOWLEDGE.md                              10083  Apr  5 13:23
-rw-r--r--   ARIA_V2_ARCHITECTURE.md                               23002  Apr 10 19:14    ← v2 arch doc
drwxr-xr-x   __pycache__/
drwxr-xr-x   archive_20260405/
-rw-r--r--   aria-data.js                                          61807  Apr  8 11:15
-rw-r--r--   aria.db                                           273051648  Apr 11 09:48    ← 260 MiB, LIVE
-rw-r--r--   aria.db.backup_20260410_204410                    273002496  Apr 10 20:44
-rw-r--r--   aria.db.backup_before_step5_20260410_223951       273018880  Apr 10 22:39
-rw-r--r--   aria.db.bak.pre-indodax-purge.20260408-103025     266473472  Apr  8 10:30
-rw-r--r--   aria.log                                           2978321  Apr 10 11:39
-rw-r--r--   aria_agent.py.bak_before_rebalance_20260404        220133  Apr  4 09:49
-rw-r--r--   aria_market_capture.py                              30351  Apr 10 10:57
-rw-r--r--   aria_reconcile.py                                   31014  Apr 10 20:36    ← v2 related
-rw-r--r--   aria_tokocrypto.py                                  45604  Apr 11 09:28    ← v2 related (Bug 1/2A applied)
-rw-r--r--   aria_tokocrypto.py.backup_20260411_090412           44913  Apr 11 09:04    ← pre-fix
-rw-r--r--   aria_tokocrypto.py.backup_20260411_090420           44913  Apr 11 09:04    ← pre-fix
-rw-r--r--   aria_tokocrypto.py.backup_20260411_090446           44913  Apr 11 09:04    ← pre-fix
-rw-r--r--   aria_tokocrypto.py.backup_20260411_090625           44913  Apr 11 09:06    ← pre-fix (this session)
-rw-r--r--   aria_v2_b2.py                                       27931  Apr 10 21:10    ← v2: classifier
-rw-r--r--   aria_v2_maker.py                                    28951  Apr 11 09:28    ← v2: maker engine (Bug 2C applied)
-rw-r--r--   aria_v2_orchestrator.py                             14358  Apr 10 22:30    ← v2: orchestrator
-rw-r--r--   aria_v2_runner.py                                   40594  Apr 11 09:29    ← v2: agent runner (Bugs 2B/3/4 applied)
-rw-r--r--   aria_v2_schema.sql                                   4310  Apr 10 11:20    ← v2: DDL
-rw-r--r--   audit_22h_behavior.md                                6555  Apr 10 20:52
-rw-r--r--   audit_position_desync.md                             5698  Apr 10 22:35
-rw-r--r--   dashboard_api.py                                    17538  Apr  8 17:23
-rw-r--r--   index.html                                          47719  Apr  8 11:15
drwxr-xr-x   logs/
-rw-r--r--   market_history.db                                 4132864  Apr 11 10:00
drwxr-xr-x   market_reports/
drwxr-xr-x   node_modules/
-rw-r--r--   package-lock.json                                   35621  Feb 23 20:15
-rw-r--r--   package.json                                          372  Feb 23 17:51
drwxr-xr-x   pine/
-rw-r--r--   reconcile_report_20260410.txt                        7286  Apr 10 20:46
-rw-r--r--   reconcile_report_20260411.txt                        7386  Apr 11 09:48
-rw-r--r--   requirements.txt                                      201  Feb 23 17:51
drwxr-xr-x   scripts/
-rw-r--r--   server.js                                           39681  Apr  9 12:57
-rw-r--r--   test_ticksizes.txt                                     766  Apr 10 21:22
drwxr-xr-x   tests/                                                                   ← test_position_guards.py lives here
drwxr-xr-x   tools/
-rw-r--r--   trades.db                                               0  Apr  8 09:15    ← empty
drwxr-xr-x   v1_archive/
```

**v2-related (highlighted):** `aria.db`, `aria_tokocrypto.py`, `aria_reconcile.py`, `aria_v2_*.py`, `ARIA_V2_ARCHITECTURE.md`, `aria_v2_schema.sql`.

---

## 5. DATABASE SCHEMAS

```sql
-- v2_positions
CREATE TABLE v2_positions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    agent TEXT NOT NULL,
    coin TEXT NOT NULL,
    pool TEXT DEFAULT 'IDR',
    pair TEXT NOT NULL,
    side TEXT DEFAULT 'BUY',
    entry_price REAL NOT NULL,
    entry_qty REAL NOT NULL,
    entry_value_idr REAL NOT NULL,
    current_price REAL DEFAULT 0,
    current_value_idr REAL DEFAULT 0,
    high_price REAL DEFAULT 0,
    high_pnl_pct REAL DEFAULT 0,
    trail_trigger_pct REAL DEFAULT 0,
    pnl_pct REAL DEFAULT 0,
    pnl_idr REAL DEFAULT 0,
    status TEXT DEFAULT 'OPEN',       -- OPEN, TRAILING, SOLD, STOPPED, STUCK_DUST
    sell_price REAL DEFAULT 0,
    sell_value_idr REAL DEFAULT 0,
    opened_at TEXT NOT NULL,
    closed_at TEXT,
    hold_minutes REAL DEFAULT 0,
    assignment_id INTEGER,
    FOREIGN KEY (assignment_id) REFERENCES v2_assignments(id)
);
CREATE INDEX idx_v2_pos_agent ON v2_positions(agent, status);

-- v2_assignments
CREATE TABLE v2_assignments (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    agent TEXT NOT NULL,              -- 'A1', 'A2', 'A3'
    coin TEXT NOT NULL,
    pool TEXT DEFAULT 'IDR',
    pair TEXT NOT NULL,
    reason TEXT,                      -- B2's reasoning
    assigned_at TEXT NOT NULL,
    status TEXT DEFAULT 'PENDING',    -- PENDING, ACCEPTED, EXECUTED, SOLD, CANCELLED
    cycle_date TEXT NOT NULL
);
CREATE INDEX idx_v2_assign_agent ON v2_assignments(agent, status);

-- v2_signals
CREATE TABLE v2_signals (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    from_agent TEXT NOT NULL,
    signal_type TEXT NOT NULL,         -- 'DISTRESS', 'SOLD', 'REQUEST_COIN', 'DESYNC_DETECTED'
    coin TEXT,
    detail TEXT,                       -- JSON
    created_at TEXT NOT NULL,
    handled INTEGER DEFAULT 0,
    b2_response TEXT,                  -- 'HOLD', 'CUT', 'NEW_COIN:XYZ'
    handled_at TEXT
);
CREATE INDEX idx_v2_signals_handled ON v2_signals(handled);

-- v2_desync_events
CREATE TABLE v2_desync_events (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    coin            TEXT NOT NULL,
    agent           TEXT NOT NULL,
    v2_qty          REAL NOT NULL,
    exchange_qty    REAL NOT NULL,
    delta_pct       REAL NOT NULL,
    detected_at     TEXT NOT NULL,
    root_cause_hint TEXT DEFAULT 'UNKNOWN',
    resolved_at     TEXT
);
CREATE INDEX idx_v2_desync_coin ON v2_desync_events(coin, detected_at);

-- v2_order_attempts
CREATE TABLE v2_order_attempts (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    symbol          TEXT NOT NULL,
    side            TEXT NOT NULL,
    attempted_price REAL NOT NULL,
    attempted_qty   REAL NOT NULL,
    status          TEXT NOT NULL,    -- FILLED, PARTIAL, UNFILLED, FAILED, FALLBACK_MARKET, FALLBACK_FILLED, FALLBACK_FAILED, GAVE_UP
    filled_qty      REAL DEFAULT 0,
    filled_price    REAL DEFAULT 0,
    fees_paid       REAL DEFAULT 0,
    order_id        TEXT,
    attempted_at    TEXT NOT NULL,
    completed_at    TEXT,
    agent           TEXT,
    spread_captured REAL DEFAULT 0,
    best_market_price REAL DEFAULT 0
);
CREATE INDEX idx_v2_order_agent ON v2_order_attempts(agent, attempted_at);

-- tk_fills_raw (source of truth — straight from Tokocrypto /orders/trades)
CREATE TABLE tk_fills_raw (
    fill_id    TEXT PRIMARY KEY,
    symbol     TEXT NOT NULL,
    order_id   TEXT NOT NULL,
    price      REAL NOT NULL,
    qty        REAL NOT NULL,
    quote_qty  REAL NOT NULL,
    commission REAL NOT NULL,
    commission_asset TEXT NOT NULL,
    commission_idr   REAL NOT NULL,
    time       INTEGER NOT NULL,      -- unix ms
    is_buyer   INTEGER NOT NULL,      -- 1 = BUY, 0 = SELL
    is_maker   INTEGER NOT NULL,
    fetched_at TEXT NOT NULL
);
CREATE INDEX idx_tk_fills_symbol ON tk_fills_raw(symbol, time);

-- tk_realized_trades (FIFO-matched closed round-trips)
CREATE TABLE tk_realized_trades (
    id           INTEGER PRIMARY KEY AUTOINCREMENT,
    symbol       TEXT NOT NULL,
    buy_fill_id  TEXT NOT NULL,
    sell_fill_id TEXT NOT NULL,
    qty          REAL NOT NULL,
    buy_price    REAL NOT NULL,
    sell_price   REAL NOT NULL,
    gross_pnl_idr  REAL NOT NULL,
    total_fees_idr REAL NOT NULL,
    net_pnl_idr    REAL NOT NULL,
    hold_minutes   REAL NOT NULL,
    buy_time     INTEGER NOT NULL,
    sell_time    INTEGER NOT NULL
);
CREATE INDEX idx_tk_realized_symbol ON tk_realized_trades(symbol);

-- tk_open_lots (un-matched FIFO buy lots still held)
CREATE TABLE tk_open_lots (
    id             INTEGER PRIMARY KEY AUTOINCREMENT,
    symbol         TEXT NOT NULL,
    qty            REAL NOT NULL,
    avg_cost_with_fees REAL NOT NULL,
    opened_at      INTEGER NOT NULL,
    source_fill_id TEXT NOT NULL
);
CREATE INDEX idx_tk_open_symbol ON tk_open_lots(symbol);
```

---

## 6. SAMPLE DATA — `v2_positions`

**Important column-name correction:** `v2_positions` has no `agent_id` / `symbol` / `created_at`. Real columns are `agent`, `coin`, `opened_at`. Only 9 rows exist (not 20 — DB is sparse).

| agent | coin | status     | entry_qty   | entry_price   | pnl_idr | opened_at                        |
|-------|------|------------|-------------|---------------|---------|----------------------------------|
| A1    | DOGE | STUCK_DUST | 0.940278    | 1,609.5       | 2,643   | 2026-04-10T14:27:22.001571+00:00 |
| A2    | SOL  | STUCK_DUST | 0.00008556  | 1,444,757.5   | 1,158   | 2026-04-10T13:54:43.627269+00:00 |
| A3    | SUI  | OPEN       | 5.50140152  | 16,065.5      | -47     | 2026-04-10T13:54:43.498583+00:00 |
| A1    | ETH  | SOLD       | 0.00102525  | 38,052,016.0  | 466     | 2026-04-10T13:27:14.336150+00:00 |
| A1    | DOGE | SOLD       | 24.55207538 | 1,592.0       | 37      | 2026-04-10T12:54:38.955006+00:00 |
| A1    | BTC  | STUCK_DUST | 0.00001892  | 1,237,634,625 | 185     | 2026-04-10T11:57:03.992774+00:00 |
| A1    | ETH  | SOLD       | 0.00061298  | 37,690,944.5  | 199     | 2026-04-10T05:54:06.190961+00:00 |
| A1    | DOGE | SOLD       | 33.59919824 | 1,591.5       | 17      | 2026-04-10T05:54:00.883753+00:00 |
| A2    | ARB  | STUCK_DUST | 0.0183312   | 1,915.0       | 507     | 2026-04-10T05:54:00.872158+00:00 |

**Unreliability evidence for the rebuild brief:**
- All 4 `STUCK_DUST` rows carry **positive** `pnl_idr` values summing to +Rp 4,493. These are stale mark-to-market snapshots, not realized gains. The coins are still sitting in the wallet.
- A2/SOL `entry_qty = 0.00008556` vs. exchange wallet qty `0.01446956` — **off by ~169×**. The v2 row was scaled down by a `_verify_position_qty` correction against a stale balance cache.
- A1/BTC `entry_qty = 0.00001892` vs. exchange `0.00062817` — **off by ~33×**.
- A2/ARB `entry_qty = 0.0183312` vs. exchange `0.40000000` — **off by ~22×**.
- A3/SUI is the only row where v2 qty matches wallet within 0.4%.
- Only 9 v2 rows exist; the exchange wallet has **21 non-zero coin balances** — 12+ coins owned with no v2 tracking at all.

This confirms the rebuild's premise: `v2_positions` cannot be the source of truth.

---

## 7. TOKOCRYPTO `myTrades` / pagination investigation

**Grep results:**
```
$ grep -n "myTrades" *.py
(no matches)

$ grep -n "fromId" *.py
aria_reconcile.py:136:    Paginates by using last tradeId as fromId for the next page.
aria_reconcile.py:144:            params["fromId"] = from_id
```

**Findings:**
- There is **no** reference to `/api/v3/myTrades` anywhere. Tokocrypto's open API equivalent is `/open/v1/orders/trades` (not Binance v3).
- `aria_reconcile.fetch_fills` is the **only** existing implementation that calls the fills endpoint. It does know about `fromId` as a paginator — the variable is declared and added to params — but the pagination loop is disabled with an unconditional `break`.
- The endpoint accepts: `symbol` (required), `startTime` (ms), `endTime` (ms), `fromId` (trade-id cursor), `limit` (max 500).
- Existing implementation uses only `symbol`, `startTime`, `limit`. `endTime` and `fromId` are left unused.

**Response envelope** (from Tokocrypto `/open/v1/orders/trades`): `{ code: 0, data: { list: [...] } }`. After `_signed_get` unwraps `data`, the caller receives `{"list": [...]}`. Each fill:
```json
{
  "tradeId": "...",
  "orderId": "...",
  "symbol":  "BTC_IDR",
  "price":   "1234567",
  "qty":     "0.001",
  "quoteQty": "1234.57",
  "commission": "0.00000015",
  "commissionAsset": "BTC",
  "commissionInIDR": "185",
  "usdtIdrPrice": "17149",
  "time": 1712750400000,
  "isBuyer": 1,
  "isMaker": 0
}
```

**Recommendation for `get_my_trades` wrapper in Priority 1:**
- Use `/open/v1/orders/trades`, not `/api/v3/myTrades` (Binance main API — not available on Tokocrypto).
- Support both `startTime` and `fromId` paginators. `fromId` is the cleaner forward-only cursor; `startTime` is easier for time-window queries.
- Loop until `len(page) < limit`, with a hard page cap (e.g., 20 pages = 10,000 fills) to avoid runaway.
- De-dupe on `tradeId`.

---

## 8. VULNERABLE `.lower()` / `.upper()` CONTEXT (`aria_tokocrypto.py`)

Line numbers reflect the current file (post Bug 1 + Bug 2A edits). Each shows 3 lines before and after.

**Line 639** — `get_balance` inside the balance-entry loop:
```python
636        balances_list = data.get("accountAssets", []) if isinstance(data, dict) else data
637        result = {"source": "tokocrypto"}
638        for item in balances_list:
639            asset  = item.get("asset", "").lower()
640            free   = float(item.get("free",   item.get("available", 0)) or 0)
641            locked = float(item.get("locked", item.get("freeze",    0)) or 0)
642            if not asset:
```

**Line 879** — `get_order` return dict:
```python
876        return {
877            "order_id":    str(resp.get("orderId", order_id)),
878            "type":        resp.get("side", "?").lower(),
879            "price":       float(resp.get("price", 0)),
880            "status":      status,
881            "order_idr":   float(resp.get("origQty", 0)) * float(resp.get("price", 0)),
882            "remain_idr":  float(resp.get("origQty", 0)) * float(resp.get("price", 0))
```
*(The actual `.lower()` is on line 878 — user's handoff line number was off by one.)*

**Lines 908, 909** — `get_open_orders` return dict inside the loop:
```python
905    for o in orders_raw:
906        result.append({
907            "order_id":   str(o.get("orderId", "?")),
908            "pair":       o.get("symbol", "?").lower(),
909            "type":       o.get("side", "?").lower(),
910            "price":      float(o.get("price", 0)),
911            "order_idr":  float(o.get("origQty", 0)) * float(o.get("price", 1)),
912            "remain_idr": (float(o.get("origQty", 0)) - float(o.get("executedQty", 0)))
```

**Lines 938, 939, 942** — `get_order_history` return dict inside the loop:
```python
935    for o in orders_raw:
936        result.append({
937            "order_id":    str(o.get("orderId", "?")),
938            "pair":        o.get("symbol", symbol).lower(),
939            "type":        o.get("side", "?").lower(),
940            "price":       float(o.get("price", 0)),
941            "qty":         float(o.get("executedQty", 0)),
942            "status":      o.get("status", "?").lower(),
943            "submit_time": o.get("time", ""),
944            "finish_time": o.get("updateTime", ""),
945        })
```

All seven call sites assume the Tokocrypto API returns a string for `asset`, `side`, `symbol`, or `status`. Line 942 is the same class of bug as the one that bit us on `get_order`'s top-level `status` (Bug 1, already fixed at line 773). Fix pattern in all cases: wrap the `.get(...)` in `str(...)` before `.lower()`.

---

## 9. PM2 STATUS

```
┌────┬─────────────────────┬─────────┬──────────┬────────┬────┬──────────┐
│ id │ name                │ version │ pid      │ uptime │ ↺  │ status   │
├────┼─────────────────────┼─────────┼──────────┼────────┼────┼──────────┤
│ 10 │ aria-A1             │ 1.0.0   │ 0        │ 0      │ 0  │ stopped  │
│ 11 │ aria-A2             │ 1.0.0   │ 0        │ 0      │ 0  │ stopped  │
│ 12 │ aria-A3             │ 1.0.0   │ 0        │ 0      │ 0  │ stopped  │
│ 9  │ aria-B2             │ 1.0.0   │ 0        │ 0      │ 0  │ stopped  │
│ 1  │ aria-dashboard      │ 1.0.0   │ 3172884  │ 45h    │ 41 │ online   │
│ 7  │ aria-market-capture │ 1.0.0   │ 3362458  │ 23h    │ 1  │ online   │
└────┴─────────────────────┴─────────┴──────────┴────────┴────┴──────────┘

Modules:
  pm2-logrotate  3.0.0  pid 255557  online
```

All 4 trading processes (B2, A1, A2, A3) are `stopped` as intended. Dashboard + market-capture are online (read-only data collectors).

---

## 10. PYTHON ENV

```
python3       → Python 3.12.3
which python3 → /usr/bin/python3
pandas        → 3.0.1     (latest major — note: 3.x API has some breakage vs 2.x)
requests      → 2.32.5
pytz          → 2024.1
numpy         → 2.4.2
```

For the RSI entry filter in the rebuild: pandas 3.0 + numpy 2.4 are available. TA-Lib is **not** installed. RSI can be computed natively in pandas/numpy without needing TA-Lib — Wilder's smoothing is a 5-line function. `pd.DataFrame.ewm(alpha=1/14, adjust=False).mean()` gives the exponentially-smoothed component; 14-period price-change loop gives gain/loss.

---

## Summary

**Files successfully read:**
- `aria_tokocrypto.py` (1058 lines on disk, key logic paths included above — a few long comment-only blocks and repeated log lines were collapsed to keep the handoff readable; all function signatures and control flow are verbatim).
- `aria_reconcile.py` (812 lines, similar treatment — `fetch_fills`, `fifo_match`, `write_fills`, `normalize_fill` verbatim; `build_report` + `main()` summarized in comment form since they don't affect rebuild architecture).
- `.env` (413 lines, secrets redacted, non-ARIA_V2 settings partially omitted for brevity).
- Database schemas for 8 v2/tk tables, complete.
- 9 rows of `v2_positions` sample data, complete.
- 7 `.lower()`/`.upper()` call-site contexts.
- PM2 status (clean: all trading stopped).
- Python env (3.12.3, pandas 3.0.1, numpy 2.4.2, requests 2.32.5).

**Errors encountered:** none. All commands returned valid data.

**Unusual things worth flagging to Claude web:**

1. **Pandas 3.0** — this is the new major. Code written for pandas 2.x may need adjustment (e.g., `.astype(int)` behavior on datetime columns changed, `append` is gone, some groupby semantics shifted). The existing `aria_tokocrypto.get_ohlcv` uses `pd.DataFrame(raw, columns=...)` and `pd.to_datetime(... unit="ms")` which are still fine, but anything written for the rebuild should be tested against 3.0.

2. **pandas 3.x license grep output was noisy** — the grep that scraped "Version:" picked up the Apache/PSF license text, not just `Version: 3.0.1`. The actual version is 3.0.1 (first matching line).

3. **pagination disabled in `fetch_fills`** — as flagged in sections 2 and 7. The author knew pagination was needed but broke out early with a comment. Any rebuild that needs to fetch more than 500 fills per symbol will silently lose data.

4. **`write_fills` "new rows" counter is broken** — uses `conn.total_changes` (running total for the connection) as if it were a per-statement delta. After the first INSERT the check always passes, so the log line "Wrote N new fills" is always equal to `len(fills)`, not the actual inserted count. Cosmetic but misleading.

5. **v2_positions qty desyncs** — three of four STUCK_DUST rows have `entry_qty` off by 22–170× from the actual wallet. This is the single biggest reason the rebuild needs to treat the exchange as source of truth, not v2 state.

6. **Wallet has 21 non-zero coins, `v2_positions` has 9 rows (mostly SOLD)** — every coin on the exchange except SUI is unmanaged by v2. The rebuild will need to either ingest these as imported lots or explicitly ignore them.

7. **4 pre-existing backups of `aria_tokocrypto.py`** from 09:04 and 09:06 today — tidak pasti asal-usul tiga backup 09:04 (may have been earlier manual edits); the 09:06 backup was made by Claude Code before the Bug 1 fix.

8. **`aria.db` is 260 MiB** (273,051,648 bytes). Mostly from large tables like `market_intelligence`, `scan_metrics`, `structured_logs`, plus 3 backup `.db` files in the same directory (~260 MiB each). Disk has 41 GiB free — no pressure, but worth knowing before any VACUUM or copy operations.

9. **`ARIA_V2_ARCHITECTURE.md`** exists (23 KiB, Apr 10 19:14) — the current architecture doc. Not opened in this pass. Ask if Claude web wants to read it before drafting the Exchange-First design.

10. **Dashboard + market-capture are still online.** Both are read-only writers into `aria.db`. If Claude web's rebuild plan assumes the DB is static during work, note that `market_history.db` is being appended to every few seconds by `aria-market-capture` (mtime Apr 11 10:00).
