Hello Again!  So not seeing a ton of responses to my questions I figured I'd follow the old internet adage: If you want good advice on the internet, first post your own bad advice -- somebody smarter will correct you, guaranteed.  So here we go...

Kishok's Guide To HyperOpt, or how I waste 10 hours a day staring at charts:

First, think about your bots strategy as a set of buy and sell "signals" that are generated from market data and used to either buy or sell real assets.  Your buy signal may be a BB calculation, or some crazy sorcery involving multiplication, either way at the end of the day it's a yes or no decision -- should I buy this asset right now or not?  In the same way your sell signal, even if that signal is stop-loss or roi, it's still a yes or no -- should I sell right now or not?

Given that, the goal of optimization is to place those signals at the most ideal points in the chart so you get your lambo as quickly as possible. HyperOpt is just one tool to help you do that, along with backtesting, dry running and looking at the live graphs yourself in frequi.  When I'm working on my strategy I'm using all these tools in concert, I never rely on a single tool to make a decision.  My broad approach is like this:

1) I have a live bot, it has a property set in json format informing it's buy/sell signal parameters.
2) I have a dry-run bot running in parallel that is testing my next possible property set.
3) I have a series of in-dev property sets that I'm currently working on but not willing to commit real time to yet.

So I'm thinking of my bot in terms of those property json files.  My current best is what I'm running live, what I think is better I let dry run for a week and then push live if it really is better.  But how does any of this relate to HyperOpt?

When I feel like generating a new property set, maybe every couple weeks or so as the market changes, I first pull all the 5m, 1h and 1m historical data for my 10 currently most profitable pairs from the live bot.  I pull 60 days of data, and generally use 60 days when I'm opting. My bot runs on a 5m timeframe with a 1h look back timeframe.  I use the 1m data for the timeframe-detail while backtesting. Now I follow this procedure:

1) I set all my parameters to be optimized and run a 2000 epoch pass on the 60 days of data I have for my 10 most profitable pairs. This takes a few hours and I sort of consider it the starting point for any property set. Think of this as like putting your strategy in a blender and pressing "I'm feeling lucky".
2) Now I run a backtest on the last 30 days, the last 20 days, the last 10 days and the last 24 hours.  Here I'm looking to see what buy signals are most active/profitable vs which are least active/profitable.  I don't care too much about the results, only how the various signals are performing.
3) I start the bot in a dry-run trade mode and compare the signals it has generated right now vs my current live bot. If the little green triangles look to me like they are in a better spot I'm happy, if not then I'm sad. I make my plots show the "buy_tag" so I can tell which signal is generating which green triangle.
4) Now I step back and think about what I just saw.  I'm trying to decide which signals need to be further optimized vs which seem ok.
5) I now enable only 1 buy signal at a time and re-opt for 1000 epochs. So if my signal 5 generated no hits in the backtesting I may choose to re-opt just that signal to see if I can "wake it up" so to speak.
6) This is time consuming but I repeat steps 2/3/4 until I'm happy with all of my buy signals performance over these time periods vs my live bot.

That will get me to a good place buy signal wise.  After doing all that I'm usually pretty happy with the plots generated and the green triangle positions. Next up is selling, my bot doesn't use the normal stop-loss but rather a custom function which behaves just like the other sell signals, I do use roi. So I do this:

1) Only enable all my sell signals to be optimized. Remember I was happy with the green triangles so I don't want to mess with those anymore.
2) Run a 2000 epoch HyperOpt on the full set of sell signal properties.
3) Run the backtesting/dry-running as I did in steps 2 and 3 of the buy optimization process.
4) Re-opt specific sell signals until I'm happy with their performance in backtesting.

At this point my backtesting should be showing me strong buy and sell signal performance, hopefully better than my current live property set or I've just wasted a lot of time. From here I will sometimes run a 1000 epoch roi optimization pass but usually I prefer to tweak that one by hand instead as I feel like optimizing it is just playing games with past results.  Generally I'd prefer things be sold based on my signals and not roi, I look at roi as sort of a fallback to prevent the bot from holding onto a dead asset for too long.

One quick note on the HyperOptLoss choices, you have many and I use them all.  Also get familiar with hyperopt-list and hyperopt-show.  I always hunt through the epoch results for my own ideal result, usually I'm looking for a higher average profit per trade, or higher number of trades or some balance of those two. When I run my big 2000 epoch passes I've been relying on Calmar.  For spot optimization of specific signals I prefer the SortinoDaily although I don't know why, it just seems to do better for specific cases.

I hope you enjoyed reading this, remember I'm just a rando from the internet and you should never listen to my advice.  I only wrote this so that smarter people could correct me.

Troubleshooting, or problems I've encountered and maybe solved?
Q) When I opt my drawdown is always --, what does this mean, why is it happening?
A) This means that epoch/run made no bad trades.  This may sound great but really it isn't realistic, something is probably wrong.  Either your stoploss is too tight so you never lose money (which conversely means you likely are not holding long enough to make money either) or your properties have overfit to the specific data you are opting on. 
Fix?) Manually expand your stoploss/roi to get a little more risk into your pool, or run on a wider set of data

Q) I ran this for 2000 epochs on a signal and nothing changed, the 1st result was still the best result
A) Your signal is either not working or already overfit for the data you are running on. 
Fix?) Manually mess with that signals parameters. If you think the max for this parameter is 1.0 change it to 10.0 and see if anything happens.  If nothing happens this signal probably isn't working at all so go debug it. If something happens then re-opt on a wider data range or reconsider your max/min optimization settings for this signal -- market conditions may have changed enough to invalidate your old values.
