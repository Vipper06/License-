//+------------------------------------------------------------------+
//| Moving Average Crossover EA for MT5                             |
//| Description: Buys when Fast MA crosses above Slow MA, Sells when opposite |
//+------------------------------------------------------------------+

// Input parameters
input int FastMAPeriod = 10;   // Fast Moving Average period
input int SlowMAPeriod = 50;   // Slow Moving Average period
input double LotSize = 0.1;    // Lot size
input double StopLoss = 50;    // Stop Loss (in pips)
input double TakeProfit = 100; // Take Profit (in pips)
input int Slippage = 2;        // Slippage
input int MagicNumber = 12345; // Unique ID for trades

// Function to get moving average values
double GetMA(int period, int shift) {
    return iMA(Symbol(), 0, period, 0, MODE_SMA, PRICE_CLOSE, shift);
}

// Check if trade conditions are met
void CheckTrade() {
    double FastMA_Current = GetMA(FastMAPeriod, 0);
    double FastMA_Previous = GetMA(FastMAPeriod, 1);
    double SlowMA_Current = GetMA(SlowMAPeriod, 0);
    double SlowMA_Previous = GetMA(SlowMAPeriod, 1);
    
    // Define stop loss and take profit levels
    double price = Ask;
    double sl = price - StopLoss * Point;
    double tp = price + TakeProfit * Point;
    
    // Check for buy condition (Fast MA crosses above Slow MA)
    if (FastMA_Previous < SlowMA_Previous && FastMA_Current > SlowMA_Current) {
        if (PositionSelect(Symbol()) == false) { // Ensure no existing trade
            MqlTradeRequest request;
            MqlTradeResult result;
            
            request.action = TRADE_ACTION_DEAL;
            request.type = ORDER_TYPE_BUY;
            request.symbol = Symbol();
            request.volume = LotSize;
            request.price = Ask;
            request.sl = sl;
            request.tp = tp;
            request.magic = MagicNumber;
            request.deviation = Slippage;
            
            OrderSend(request, result);
        }
    }
    
    // Check for sell condition (Fast MA crosses below Slow MA)
    price = Bid;
    sl = price + StopLoss * Point;
    tp = price - TakeProfit * Point;
    
    if (FastMA_Previous > SlowMA_Previous && FastMA_Current < SlowMA_Current) {
        if (PositionSelect(Symbol()) == false) {
            MqlTradeRequest request;
            MqlTradeResult result;
            
            request.action = TRADE_ACTION_DEAL;
            request.type = ORDER_TYPE_SELL;
            request.symbol = Symbol();
            request.volume = LotSize;
            request.price = Bid;
            request.sl = sl;
            request.tp = tp;
            request.magic = MagicNumber;
            request.deviation = Slippage;
            
            OrderSend(request, result);
        }
    }
}

// Main loop
void OnTick() {
    CheckTrade();
}
