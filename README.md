# codingame-LOCM-postmortem
Postmortem for the "Legends of Code and Magic" competition held on codingame.com

Link to the competition results.


Introduction.
Legends of Code and Magic (LOCM for short thereafter) is based on the popular cards games (think Hearthstone and The Elder Scroll Legends). This is a two-player game. The game consists of two phases - draft phase and battle phase. During draft phase players go through thirty rounds, where in each round player has to choose one card out of three offered. By the end of the draft phase each player has a deck of 30 cards. Both players start with manapool of one mana crystal. Manapool  increases on each turn until it reaches twelve mana crystals and stays at this level during the rest of the game.Each turn players can spend mana crystals to summon creatures or use items (items are like spells in Hearthstone). Players also can also use their creatures on the board to attack enemy creatures or enemy hero. The player whose health becomes non-positive (that is, less than or equal to zero first) first, loses the game. LOCM also has rune mechanics - each player has 5 runes at the start of the game (rune number i corresponds to healh level 5 * i). After player's healh goes below the rune threshold for the first time in the game, the corresponding rune is destroyed and player will draw one card per destroyed rune next turn. If the rune is destroyed when there are no cards left in the deck, player's health decreases to the corresponding rune's threshold (if there are no runes left and you have to draw a card, you lose the game immediately). This is a bried summary of the LOCM rules to give an idea of what the game was like.


Draft phase.
I have some experience in CCGs (played Hearthstone for about a year and a half), so I decided first to assign each card a draft score based on my understanding and experience in cards games. After all scores have been assigned, the bot just had to pick the best card out of three in every round of draft phase. I was more or less OK with my draft, but still was not sure whether it was good enough. Meanwhile, the first week of the contest was over and around that the new leader emerged (aka ClosetAI), who (intentionally?) leaked his draft card valuations - just by printing them during the draft phase. Of course, the draft values of the leader were immediately studied by other people with predictable results. Many people (myself included) first literally copied all the values. I suspect that most of the people who borrowed the draft saw immediate winrate increase solely due to the new draft values. The general idea of ClosetAI's draft was simple and I must say that my own draft was rather similar, but there were some differences. In particular, I gave higher values to creatures with good stats (like 4/5 creature for 4 mana - the analogue of the famous Chillwind Yeti card from Hearthstone which was basically a no-brainer in arena drafts, because it was too powerful for its manacost). At the same time, I underestimated the value of creatures with ward. One reason for this could be my Hearthstone background. Ward (aka divine shield) in Hearthstone is in my opinion much weaker than it is in LOCM, because in Hearthstone there are more ways to remove ward (e.g., mage's and druid's heropower, weapons, misc spells). And in LOCM ward creatures oftern trade two- (if not three-) for-one, simply because you need to use one creature or item to remove the ward and second creature or item to kill the creature. As a result, almost all creatures with ward ability in LOCM are extremely strong if not overpowered. After I incorporated ideas from ClosetAI's draft into my own drafting strategy, I was trying to improve it further, but not to a big success. I tried to incorporate balancing-the-manacurve ideas into my draft strat, but the results were inconclusive - even after thousands games of self-play the winrate was oscillating in the neighbourhood of 50%. I also tried to set maximum limit of particular cards (like no more than one silence card per deck, no more than one Possesed skull etc.), which kinda worked so I kept it. During the rest of the contest I was mostly tuning the draft by watching the replays and trying to understand wheter the loss was caused by bad draft of wrong choices during the battle phase.


Battle phase.
The battle phase of the game may seem complex at first, especially if you are not familiar with card games. But after couple of dozens of replays the idea becomes clear - try to make value trades by using the resources available (creatures on the boards and cards in hand). The ideal scenario is when you kill enemy high-value creature with you low-value creature. If you are doing it during several consecutive turns, you will notice than you are slowly but surely gaining control over the board. This showballing effect is extremely hard to stop. One of the main reasons is the lack of comeback mechanisms (like board-wipe with AOE spell or one-turn combo to kill the opponent hero). If both players are making good trades and use their resources wisely, the outcome of the game is often decided by the draw order and/or draft choices (sometimes even one card difference during draft phase can decide the outcome of the game).
I decided to implement simulation of my and opponents' actions. I wrote the simulation from scratch based only on the rules of the game and not using the referee code (probably it was a mistake, because the referee source code can contain some ideas on how to model the game state, board, move generation etc - the things that might be of great use when trying to implement your own simulation). The simulation was based on the brute-force generation of all possible moves of my player and all possible attacks of enemy player. In order to handle the combinatorial explosion on busy turns resulting in huge number of tree nodes to explore, I decided to hash the positions using something similar to Zobrist hashing - a common technique used in board games programming (think chess, checkers, othello etc). After I implemented hashing, I still had very rare timeouts on busy turns, but timeouts were rare enough to concentrate on something else instead. Another useful optimization was switching from computationally expensive data types like std::string in C++ to bitboards (for instance, creature abilities like ward and guard were represented as an array of integers and in order to set/remove the ability you only need to toggle a single bit, also you avoid expensive copying of std::string when cloning the game state for the next simulation turn). Most of my time during this contest was spent on thinking about the evaluation function. I rather quickly came up with a decent eval function that was the sum of creatures' attack and defense with positive bonuses for abilities like ward, lethal, card draw etc. I also hardcoded several cards with high value (like Decimate and Possessed Abomination) to make sure they are played in the proper moment. After this I spent a lot of time trying to tune the coefficients of the eval function by doing self-play of versions of my bot with different parameters. The problem here was that even after several hundreds or couple of thousands games the winrate often was extremely close to 50/50. So I decide to tune the parameters based on my card game experience and common sense. I was rather happy with my eval function, but it still feels like it could have been improved further. Finally, I decided to implement the rune mechanics. I tried versions with and without playing around runes and the rune-aware version was playing slightly better, so I sticked with it. By rune-aware I mean playing around opponent runes - not triggering them by hitting opponents face (hence giving him free cards) and just waiting and building the board till the critical mass of my creatures was on it. If this snowballing strategy succeeds it results in a win with a high probability. Of course, it is not always possible to win this way - sometimes the opponent will rush you and kill you before you build a decent board or you can just get wrecked by a poor draw sequence. Nevertheless, I decided to use the rune-aware version for the final and it performed well.



Conclusion.
I really enjoyed the contest. It was my first AI-related competiton where I reached top ten. The previous best result was fourteenth place in Halite-2 competition which I also enjoyed a lot. It was my first experience in writing a card-game bot and I liked it. After the contest I even started to think about writing a bot for some other card game (like Hearthstone, TESL etc.) Probably LOCM was a bit too simple and straightforward in terms of gameplay. There were no board-clear spells or other comeback mechanisms available (which could make the game more deep with some mindgames involved), you just had to pick good cards during draft and make value-trades during battle phase. But still I think that the skill cap was not reached even by the best bots. I am sure that human player should be able to defeat the best LOCM bots given enough time to study and practice the game. So I cannot say that the game was too simple and there is nothing left to improve and learn about it. I know that many people were complaining about the randomness iN LOCM, but it's a card game and randomness (the card pool available for draft and order of draws) is its inherent feature. Despite presence of randomness, the final ranking reflected the pre-recalc ones pretty closely and the winner finished with a good margin above second place. The same applies to top-4 where all bots were separated from each other by a decent gap. At the same time, bots in the range 5-10 were all pretty close to each other and every bot from this range had more or less equal chances of being top-5 or top-10. Per the duration of the contest I think it was OK, but it could be shorter - probably two weeks was a better choice. Personally I enjoy longer contests more for several reasons. First, people have more time to think about the problem which should result in higher quality of the bots and deeper understanding of the game. Second, some participants can be extremely busy during one-week contests and thus are automatically excluded from it while during longer contests more people can join and create at least a  decent bot. Third, there are a lot of sprint contests nowadays (ACM-ICPC format-like that are held on codeforces.com and other similar sites) while there are not so many longer contests (at least to the best of my knowledge).

Thanks for reading and happy coding!




