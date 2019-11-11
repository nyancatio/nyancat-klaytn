**NyanCat.io smart-contract**

The contract is developed using Solidity v.0.5.6 specially for Klaytn Horizon bApps competition. 

Try live version of NyanCat.io game here – https://nyancat.io  
Check out current Waves implementation stats – https://www.dappocean.io/dapps/nyancat 

**Contract logic description**
Race scheme describes how the race will be held and what prizes will be awarded at the end. Each scheme has its unique ID, the value identifying the number of players, and an array with multipliers *100 in order of priority.
For example, coefficient 1.5 = 150, and so on.  

The server creates the race by calling the method create(uint_raceId, uint _bet, uint _duration, uint _schemeId)
Only the server can call this.
Also, the valid scheme ID, which was created earlier, needs to be transferred. No duplicate calls are allowed!
IMPORTANT:_bet must be written as peb, meaning that 1 KLAY = 10^18 peb.

After this, the server has 2 options:

1.	There’s sufficient number of players — the server starts the race, calling the method start(uint _raceId, uint _time), which receives the race ID and time (server time stamp in seconds).
Only the server can call.

2.	There’s an insufficient number of players — the server calls the method cancel(uint _raceId), which receives the race ID.
After the cancellation, users can collect back their bets by calling the method revokeBet(uint _raceId, bytes calldata _signature), which receives the race ID and server signature, concurring that the player can cancel their bet. This method can be called only if the race was cancelled.
The player can call this method for a race only once — it won’t work more than once. In our case, the method pings the server through the user contract that it generates for the player.
The server can monitor successful bet cancelation by player, the event PlayerBetRevoked(_raceId, msg.sender).
The player pays the commission for canceling the race.

After the bet was created and before it starts, players can make their bets, receiving the server signature ahead of time.
The number of players that will take part in the race must equal the number stated in the scheme, with which the race started. If the number of players equals the number stated by the race scheme and someone else tries to make a bet, the bet won’t go through.
The bet is realized as a playable method bet(uint _raceId, bytes calldata _signature), which needs to receive the necessary amount of currency, provided by the race.
Each player can make only one bet per race. 
For players to make bets, the race must be neither started nor cancelled. With that, the server can monitor the event PlayerBetPlaced, which will receive the race ID and the address of the player that made the bet.

3.	After the race has started, the server calculates when it needs to end, and ends the race in the interval not greater than duration (if this is equal to or greater than duration, it’s all right, but that will go to BC, which can then be checked), by calling the method finish(uint _raceId, uint _time, address payable[] calldata _winners), passing on the names of winners for places: 0-1, etc.


After the race has ended, the awards are calculated and transferred to the players. The difference between the total race bet and the total reward is transferred to the destination wallet.
For it to be completed, the race must be started, not yet finished and not cancelled, and a valid destination wallet must be created.
