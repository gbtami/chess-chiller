# Chess-Chiller
Read pgn files, parse positions in the game and save interesting positions for problem-solving sessions.

### A. Requirements
* Python 3 
* Python-Chess 
* Chess engines that supports multipv and movetime commands 
* PGN file

Python 3 can be found at https://www.python.org/downloads/ \
Python-chess can be found at https://github.com/niklasf/python-chess \
Chess engine that supports multipv and movetime is Stockfish. Download it from https://stockfishchess.org/download/ \
PGN file which contain game records can be downloaded from http://theweekinchess.com/

### B. Command line
`python chess-chiller.py --inpgn aeroflotopa19.pgn --engine sf10.exe --threads 1 --hash 128`\
Interesting positions will be saved in interesting.epd

If your pgn filename has space say 'my blitz games.pgn', enclose it in double quotes.\
`python chess-chiller.py --inpgn "my blitz games.pgn" --engine sf10.exe --threads 1 --hash 128`

### C. Options and flags
#### --pin
A flag used to saved only those positions when there is a piece of the side not to move that is pinned.\
`python chess-chiller.py --inpgn aeroflotopa19.pgn --engine sf10.exe --pin`

#### --skipdraw
A flag used to ignore games with draw results. Useful when you are only interested on generating positions from games with 1-0 or 0-1 results.\
`python chess-chiller.py --inpgn aeroflotopa19.pgn --engine sf10.exe --skipdraw`

#### --engine [uci chess engine]
An option to set which uci engineto use. Place this chess engine in the same directory as the chess-chiller.py.\
`python chess-chiller.py --inpgn aeroflotopa19.pgn --engine sf10.exe`

If your engine is in a different directory say c:\engines\stockfish\sf10.exe\
`python chess-chiller.py --inpgn aeroflotopa19.pgn --engine "c:\engines\stockfish\sf10.exe"`

In this repo there is Stockfish 10 engine in Engine directory. You can use it with chess-chiller. Those exe files are from https://stockfishchess.org/download/. You can also find the source codes on that link.

#### --maxtime [time in sec]
An option to allow the engine to search at a maximum time in seconds.\
`python chess-chiller.py --inpgn aeroflotopa19.pgn --engine sf10.exe --maxtime 20.0`

#### --mintime [time in sec]
An option to limit the engine search time in seconds. By default the engine is set to search at a maximum time via --maxtime value. However during the search, if the best score of the position is only -500 or lower after the minimum search time and the mimum score threshold is 100, then the program will abort engine search to save analysis time.\
`python chess-chiller.py --inpgn aeroflotopa19.pgn --engine sf10.exe --mintime 5.0`

#### --minpiecevalue [value]
An option used to control the number of pieces (not kings and not pawns) remaining on the board for saved positions. Default value is 0. The maximum is 62 or 2*Q + 4*R + 4*B + 4*N, where Q=9, R=5, B=3 and N=3. If you want middle phase positions, you may use for example 2Q + 4R + 2B + 2N or 50. Any positions with less than 50 piece value will not be saved.\
`python chess-chiller.py --inpgn aeroflotopa19.pgn --engine sf10.exe --minpiecevalue 50`

If you want interesting positions to be saved without this piece value restriction, just use\
`python chess-chiller.py --inpgn aeroflotopa19.pgn --engine sf10.exe`

#### --maxpiecevalue [value]
An option used to control the number of pieces (not kings and not pawns) remaining on the board for saved positions. Default value is 62. If you want ending positions where the maximum piece value total from both sides is 10 or 2 rooks or less use --maxpiecevalue 10.\
`python chess-chiller.py --inpgn aeroflotopa19.pgn --engine sf10.exe --maxpiecevalue 10`

#### --log [value]
An option used to save logs to all.log file. value can be **debug, info, warning, error and critical**, default is critical. If you want to see all the logs including the engine analysis, use value debug. Error messages will be saved in error.log file.\
`python chess-chiller.py --inpgn aeroflotopa19.pgn --engine sf10.exe --log debug`

#### --minbs1th1 [value]
An option called minimum best score 1 threshold 1. Default is 2000 cp. This is used to control the best score 1 from multipv 1 of the engine analysis. In order for the position to be interesting, the bs1 (best score 1) from multipv 1 of engine analysis should not be lower than minbs1th1. With that default value of 2000 cp or around 2 queens advantage, we are looking for positions that is winning. This option is useful when used together with the option maxbs2th1, see the following option. See also section E.6 on how this is used.

Example command line.\
`python chess-chiller.py --inpgn WorldBlitz2018.pgn --engine stockfish_10_x64.exe --minbs1th1 1000`

#### --maxbs2th1 [value]
An option called maximum best score 2 threshold 1. Default is 300 cp. This is used to control the best score 2 from multipv 2 of the engine analysis. In order for the position to be interesting, the bs2 (best score 2) from multipv 2 of engine analysis should not be higher than maxbs2th1. This is used together with minbs1th1. Example give a position we analyze it with an engine searching for best move at multipv 2. If bs1 is 2500 and bs2 is 200 that means we can consider this position as interesting. See also section E.6 on how this is used.\
`if bs1 >= minbs1th1 and bs2 <= maxbs2th1 then save this position`

Example program epd output.\
`8/4k3/2K4p/4p2P/2P5/8/8/8 w - - bm Kd5; ce 31980; sm Kd5; acd 40; acs 15; fmvn 76; hmvc 0; pv Kd5 e4 Kxe4 Kf6 c5; c0 "Carlsen, Magnus - Popov, Ivan RUS, World Blitz 2018, St Petersburg RUS, 2018.12.29, R1.1"; c1 "Complexity: 1"; c2 "bestscore2: 0"; c3 "Analyzing engine: Stockfish 10 64 POPCNT";`

![](https://i.imgur.com/vSbXpVU.png "White to move")

In the output epd, there are 2 scores that are important to us.
1. ce 31980;
2. c2 "bestscore2: 0";

That ce and c2 are called opcodes. ce is centipawn evaluation and c2 is comment2.\
bs1 = ce = 31980\
bs2 = bestscore2 = 0

bs1 is the best score from multipv 1 of engine analysis.\
bs2 is the best score from multipv 2 of engine analysis.

Our options are:\
minbs1th1 = 2000\
maxbs2th1 = 300

Since bs1 is greater than or equal to minbs1th1 and bs2 is less than or equal to maxbs2th1 then we can consider this position as interesting.

#### --minbs1th2 [value]
An option called minimum best score 1 threshold 2. Default is 1000 cp. This is used to control the best score 1 from multipv 1 of the engine analysis. In order for the position to be interesting, the bs1 (best score 1) from multipv 1 of engine analysis should not be lower than minbs1th2. With that default value of 1000 cp or around 1 queens advantage, we are looking for positions that is winning. This option is useful when used together with the option maxbs2th2. See also section E.6 on how this is used.

Example command line.\
`python chess-chiller.py --inpgn WorldBlitz2018.pgn --engine stockfish_10_x64.exe --minbs1th2 700`

#### --maxbs2th2 [value]
An option called maximum best score 2 threshold 2. Default is 200 cp. This is used to control the best score 2 from multipv 2 of the engine analysis. In order for the position to be interesting, the bs2 (best score 2) from multipv 2 of engine analysis should not be higher than maxbs2th2. This is used together with minbs1th2. See also section E.6 on how this is used.

#### --minbs1th3 [value]
An option called minimum best score 1 threshold 3. Default is 500 cp. This is used to control the best score 1 from multipv 1 of the engine analysis. In order for the position to be interesting, the bs1 (best score 1) from multipv 1 of engine analysis should not be lower than minbs1th3. With that default value of 500 cp or around 1 rook advantage, we are looking for positions that is winning. This option is useful when used together with the option maxbs2th3. See also section E.6 on how this is used.

Example command line.\
`python chess-chiller.py --inpgn WorldBlitz2018.pgn --engine stockfish_10_x64.exe --minbs1th3 300`

#### --maxbs2th3 [value]
An option called maximum best score 2 threshold 3. Default is 100 cp. This is used to control the best score 2 from multipv 2 of the engine analysis. In order for the position to be interesting, the bs2 (best score 2) from multipv 2 of engine analysis should not be higher than maxbs2th3. This is used together with minbs1th3. See also section E.6 on how this is used.

### D. Output
An example output epd would look like this.

`4rbrk/5Rpp/p3b3/1p1B4/4p3/1P2P3/PB6/2K3R1 w - - bm Rxf8; ce 323; sm Rxf8; acd 28; acs 20; fmvn 28; hmvc 0; pv Rxf8 Rexf8 Bxe6 h6 Bxg8; c0 "Cheparinov, Ivan - Zeng, Chongsheng, TCh-CHN 2018, China CHN, 2018.04.11, R1.2"; c1 "Complexity: 2"; c2 "bestscore2: -28"; c3 "Analyzing engine: Stockfish 10 64 POPCNT";`

The board image from that epd.

![](https://i.imgur.com/0x41SJp.png "From chess-chiller epd output, white to move!")

### E. Process flow
1. The script will read the pgn file given from --inpgn [user pgn file].
2. Will read each game in the pgn file.
3. Will parse the moves in the game in reverse. If the game has 40 moves, it will visit first the end position at 40th move, 39th move 38th, 37th and so on.
4. In every position visited the chess engine specified in --engine [engine] will be run at multipv 2 at a given maxtime from --maxtime [time in sec]. We are interested on saving the 1st best score from multipv 1 call it (bs1) and the 2nd best score from multipv 2 call it (bs2).
5. Basically when the engine shows that the side to move has a decisive advantage say bs1 >= 300 cp (centipawn) and bs2 is not winning say bs2 <= 50 cp then the program will save that position in interesting.epd file. Take a look at the example epd output in section D. The ce (centipawn evaluation) has a value of 323. That is actually bs1. And that of bs2 is at c2 "bestscore2: -28"; in this case bs2 is -28 cp.
6. The program has 6 score thresholds which will control if the position is saved or not. These are:\
minbs1th1 and maxbs2th1\
minbs1th2 and maxbs2th2\
minbs1th3 and maxbs2th3\
\
They are used in pairs and should follow the scoring hierarchy.\
minbs1th1 > minbs1th2 > minbs1th3 or minbs1th1 is greater than minbs1th2 and minbs1th2 is greater than minbs1th3.\
maxbs2th1 > maxbs2th2 > maxbs2th3\
\
Example situation 1:\
You want to save positions where the side to move is not losing but it is not winning either. You could imagine the bs1 is not below -50 cp (not a losing score) and bs2 is not above +50 cp (not a winning score). Set the following option values.\
\
minbs1th2 = +51\
minbs1th3 = -50\
Then set the limit of bs2, typically it should be around 100 cp below minbs1th3, and maxbs2th2 > maxbs2th3.\
maxbs2th2 = -50\
maxbs2th3 = -150\
\
So given bs1 and bs2 from engine analysis we can say.\
`if bs1 >= minbs1th3 and bs1 < minbs1th2 and bs2 <= maxbs2th3 then save this position.`\
\
Example analysis session:\
command line\
`python chess-chiller.py --inpgn "tatagpa19.pgn" --engine "stockfish_10_x64_popcnt.exe" --threads 1 --hash 128 --mintime 3 --maxtime 10 --minbs1th2 51 --minbs1th3 -50 --maxbs2th3 -150 --maxbs2th2 -100`\
\
EPD output:\
`8/8/8/4Nn1p/2K5/6k1/8/8 w - - bm Ng6; ce -9; sm Ng6; acd 40; acs 10; fmvn 82; hmvc 1; pv Ng6 Ng7 Kd3 Ne6 Ke4; c0 "Rapport, Richard - Shankland, Samuel, 81st Tata Steel GpA, Wijk aan Zee NED, 2019.01.13, R2.4"; c1 "Complexity: 0"; c2 "bestscore2: -6015"; c3 "Analyzing engine: Stockfish 10 64 POPCNT";`\
\
![](https://i.imgur.com/VhgIXFc.png "white to move")\
bs1 = -9\
bs2 = -6015\
minbs1th2 = 51\
minbs1th3 = -50\
maxbs2th3 = -150<br>
Since bs1 >= minbs1th3 and bs1 <= minbs1th2 and bs2 <= maxbs2th3, we save this position to interesting.epd file.\
\
Currently the program has 3 ways of saving interesting positions.\
It can save position with high score, medium score and low score. High score is controlled by minbs1th1/maxbs2th1, the medium is minbs1th2/maxbs2th2 and the low score minbs1th3/maxbs2th3.\
\
Example situation 2:\
You want to save a position where the side to move has a very advantageous position, set minbs1th1 to 3000 or 3 queens and maxbs2th1 at 500. Then at the medium set minbs1th2 at 500 and maxbs2th2 at 200 and finally at low score, set minbs1th3 at 100 and maxbs2th3 at 0.\
\
If bs1 = 5000 and bs2 = 400, it will be save because it is under the high score threshold.\
`if bs1 >= 3000 and bs2 <= 500, save this position.`\
\
However if bs1 = 1000 and bs2 = 150, it will be saved under the medium score threshold.\
`if bs1 >= 500 and bs2 <= 150, save this position.`\
\
If it is not saved under the conditions of high score and medium score thresholds, then it will be tested under the low score threshold. The low score threshold is threshold 3 or minbs1th3/maxbs2th3.

7. There are some enhancements to save the position or not, one of those is, if the side to move is in-check, don't save such position. Another one is if the best move is a capture and this position is not complicated according to the analyzing engine, such position is also not saved.

