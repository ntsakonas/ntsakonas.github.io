---
layout: post
author: Nick
date: 2020-11-22 14:28:00 -0000
---

As I was enjoying my usual archive.org browsing last night, I came across a book that contains type-in games for [Atari](https://en.wikipedia.org/wiki/Atari_8-bit_family), one of the earliest home computers. Type-in programs are a big part of my childhood and I cannot resist reading them even today!

<!--more-->
If you were a kid in the 80's it was very common to type-in your programs and games from magazines or books. 
It was a really educational process as well as a cheap way to enjoy many hours of playing games without graphics at all.
The main drawback was that most of the books did not have a single screenshot of the program, so you did not know what kind of game you would be typing for the next few days! I can recall all the dissapointing moments after running the game...

The interesting part of the book I came across is that it contains a game, that I happened to have played 
(after having typed it in of course) and really enjoyed on a different home computer (a [C64](https://en.wikipedia.org/wiki/Commodore_64)) when I was a kid!

The book is called _Stimulating Simulations, 2nd Edition, by C.W.Engel (c) 1979_ and can be found in [acrhive.org](https://archive.org/details/ataribooks-stimulating-simulations-atari-version/) and the game is called "Monster Chase".
You are locked in a cage with a hungry monster who has life span of 10 turns. The movements take place on a 5x5 grid. 
You can move east,west, south and north by entering E,W,S,N or O to remain in the same place. The monster can also move diagonally.

The goal of the game is to remain alive after 10 turns!

I thought it would be fun to follow up with that and play it once more (and not only that!). The listing is really short and I
 can now type it 20x faster that when I was 11!

The book has a few challenges for the reader in the form of suggested modifications,
which were unknown to me when I first came across the listing in a home computer magazine. 
Coming across the game after all these years, triggered a few ideas I would like to try, definitely _not_ the same I would have tried back then _if_ I had seen the challenges.

As a tribute to the era and the machine, I will keep it "BASIC" and not write fancy code, just like in 1979!

BASIC has global variables, functions and subroutines which will be emulated by members and methods if required. 
No object oriented code, just _int_, _float_, and _string_. The only exceptions will be when BASIC specific functions need to be emulated (as required).
At least I will try to keep it retro!


### Let's have fun!

The original listing from the book is copied here. 

```
M0NSTER CHASE PROGRAM 

 Variables:                                                                      
 - R,C Your row and column                                                       
 - X,Y Monster's row and column                                                  
 - L,M Temporary variables                                                       
 - M$ Your move (N,E,S,W,O)                                                      
 - D Direction of the monster (1-8)                                              
 - T Turns (1-10)                                                                
                                                                                 
 5 REM SET CONDITIONS                                                            
 10 X=1: Y=1                                                                     
 20 R=5: C=5                                                                     
 30 FOR T=1 TO 10                                                                
 35 REM DISPLAY GRID                                                             
 40 FOR I=1 TO 5                                                                 
 50 FOR J=1 TO 5                                                                 
 60 PRINT TAB(8)                                                                 
 70 IF I=X AND J=Y THEN PRINT "M";: GO TO 100                                    
 80 IF I=R AND J=C THEN PRINT "Y";: GO TO 100                                    
 90 PRINT ".";                                                                   
 100 NEXT J                                                                      
 110 PRINT                                                                       
 120 NEXT I                                                                      
                                                                                 
 210 ?:?:? "MOVE NUMBER"; T                                                      
 220 INPUT "DIRECTION (NESWO)"; M$                                               
 240 IF M$="N" THEN R=R-1                                                        
 250 IF M$="E" THEN C=C+1                                                        
 260 IF M$="S" THEN R=R+1                                                        
 270 IF M$="W' THEN C=C-1                                                        
                                                                                 
 280 IF R*C=0 OR R>5 OR C>5 THEN PRINT "OUT OF BOUNDS": GO TO 520                
 290 IF R=X AND Y=C THEN PRINT "EATEN": GO TO 520                                
 300 IF X=R AND Y<C THEN D=1                                                     
 310 IF X>R AND Y<C THEN D=2                                                     
 320 IF X>R AND Y=C THEN D=3                                                     
 330 IF X>R AND Y>C THEN D=4                                                     
 340 IF X=R AND Y>C THEN D=5                                                     
 350 IF X<R AND Y>C THEN D=6                                                     
 360 IF X<R AND Y=C THEN D=7                                                     
 370 IF X<R AND Y<C THEN D=8                                                     
 380 D=D+INT(3*RND(1)-1)                                                         
 390 IF D=0 THEN D=8                                                             
 400 IF D=9 THEN D=1                                                             
 410 IF D>1 AND D<5 THEN X=X-1                                                   
 420 IF D>5 THEN X=X+1                                                           
 430 IF D>3 AND D<7 THEN Y=Y-1                                                   
 440 IF D<3 OR D=8 THEN Y=Y+1                                                    
 450 IF X=O THEN X=X+1                                                           
 460 IF Y=O THEN Y=Y+1                                                           
 470 IF X=6 THEN X=X-1                                                           
 480 IF Y=6 THEN Y=Y-1                                                           
 490 IF X=R AND Y=C THEN PRINT "EATEN": GO TO 520                                
 500 NEXT T                                                                      
 510 PRINT "YOU SURVIVED!"                                                       
 520 INPUT "PLAY AGAIN"; Y$                                                      
 530 IF Y$="Y" THEN RUN                                                          
 540 END                                                                         
```

The program does not need a lot of introductions or explanations, it is pretty BASIC, but 2 points need a few words.

The monster can move diagonally according to which direction it sees us, but it
does not coming directly to us, as it chooses at random its next movement that wil bring it closer to us. 
This makes the game interesting and dynamic and you cannot end up with a choreography, thus, the endless fun with such a simple game!

The monster's detection of our direction is saved in variable `D` and it is translated as in the following picture

![]({{ site.url }}/images/monster_chase_monster_directions.svg)

if the player is located to the East of the monster `D` becomes 1. if the player is NorthEast of the monster `D` becomes 2 etc. 


When porting old programs, functions like `RND()` or `ROUND()` must be  looked up in the manual so that the same behaviour is replicated. 
On Atari BASIC, according to the [manual](http://www.atarimania.com/documents/Atari-Basic-Reference-Manual-Rev-C.pdf), the `RND()` works as follows:

```
RND
Format: RND(a)
Example: 10 A=RND (0)

Returns a hardware generated random number between 0 and 1, but never returns 1. 
The variable or expression in parentheses following RND is a  dummy and has no effect on the numbers returned.
However, the dummy variable must be used.

Generally, the RND function  is  used  in  combination with  other  BASIC statements or functions to return a number for  games, decision making, and the like. 
The following is a simple routine that returns a random number between 0 and 999

10 X = RND(0)
20 RX = INT(1000*X)
30 PRINT RX

0 is the dummy variable
```

so in the original listing, in line 

```
380 D=D+INT(3*RND(1)-l)
```

the possible values from RND are 0,1,2 which are adjusted to -1,0,1

so let's quickly port it to Java and play!

### Java monster chasing 

The program is ported to Java maintaining the exact structure and variable names, with minimal additions mostly 
to support the equivalent of GOTO's and the fact that the BASIC program can re-RUN it self (good old times!).

Here is the Java version, which is very "ugly" by today's software engineering standards, but let's enjoy its simplicity and assume we are in 1979!

```java
public class MonsterChaseOriginal {
    public static void main(String[] args) throws Exception {

        // simulating system input
        Scanner INPUT = new Scanner(System.in);

        // the original program can "run" it self from start
        // the closest we can go is to have the program in a do/while loop
        char Y$;
        do {
            int R, C;
            int X, Y;
            char M$;
            int D = 0;
            int T;


            // REM SET CONDITIONS
            X = 1;
            Y = 1;
            R = 5;
            C = 5;

            for (T = 1; T <= 10; T++) {
                for (int I = 1; I <= 5; I++) {
                    for (int J = 1; J <= 5; J++) {
                        System.out.print("\t\t");
                        if (I == X && J == Y) System.out.print("M");
                        else if (I == R && J == C) System.out.print("Y");
                        else System.out.print(".");
                    }
                    System.out.println();
                }

                System.out.println();
                System.out.println();
                System.out.println();
                System.out.println("MOVE NUMBER " + T);
                System.out.print("DIRECTION (NESWO) ");
                M$ = INPUT.nextLine().charAt(0);

                if (M$ == 'N') R = R - 1;
                if (M$ == 'E') C = C + 1;
                if (M$ == 'S') R = R + 1;
                if (M$ == 'W') C = C - 1;

                if (R * C == 0 || R > 5 || C > 5) {
                    System.out.println("OUT OF BOUNDS");
                    break;
                }
                if (R == X && Y == C) {
                    System.out.println("EATEN");
                    break;
                }
                if (X == R && Y < C) D = 1;
                if (X > R && Y < C) D = 2;
                if (X > R && Y == C) D = 3;
                if (X > R && Y > C) D = 4;
                if (X == R && Y > C) D = 5;
                if (X < R && Y > C) D = 6;
                if (X < R && Y == C) D = 7;
                if (X < R && Y < C) D = 8;

                D = D + (int) (Math.random() * 3.0 - 1.0);
                if (D == 0) D = 8;
                if (D == 9) D = 1;
                if (D > 1 && D < 5) X = X - 1;
                if (D > 5) X = X + 1;
                if (D > 3 && D < 7) Y = Y - 1;
                if (D < 3 || D == 8) Y = Y + 1;
                if (X == 0) X = X + 1;
                if (Y == 0) Y = Y + 1;
                if (X == 6) X = X - 1;
                if (Y == 6) Y = Y - 1;
                if (X == R && Y == C) {
                    System.out.println("EATEN");
                    break;
                }

            }
            // this is an added check as we do no have GOTO's
            // this is line 510 modified
            if (T > 10)
                System.out.println("YOU SURVIVED!");
            System.out.print("PLAY AGAIN ");
            Y$ = INPUT.nextLine().charAt(0);
        } while (Y$ == 'Y');

    }

}
```

the project (along with all future additions) can be found in [GitHub](https://github.com/ntsakonas/MonsterChase) as an Intellij project.
The code will be used as the starting point for future modifications. 


### Run Run Run!!!!

Here is the first successful escape from the monster (yes, I got eaten many times!)

```
        M       .       .       .       .
        .       .       .       .       .
        .       .       .       .       .
        .       .       .       .       .
        .       .       .       .       Y



MOVE NUMBER 1
DIRECTION (NESWO) W
        .       M       .       .       .
        .       .       .       .       .
        .       .       .       .       .
        .       .       .       .       .
        .       .       .       Y       .



MOVE NUMBER 2
DIRECTION (NESWO) W
        .       .       .       .       .
        .       .       M       .       .
        .       .       .       .       .
        .       .       .       .       .
        .       .       Y       .       .



MOVE NUMBER 3
DIRECTION (NESWO) W
        .       .       .       .       .
        .       .       .       .       .
        .       M       .       .       .
        .       .       .       .       .
        .       Y       .       .       .



MOVE NUMBER 4
DIRECTION (NESWO) E
        .       .       .       .       .
        .       .       .       .       .
        .       .       M       .       .
        .       .       .       .       .
        .       .       Y       .       .



MOVE NUMBER 5
DIRECTION (NESWO) W
        .       .       .       .       .
        .       .       .       .       .
        .       .       .       .       .
        .       .       M       .       .
        .       Y       .       .       .



MOVE NUMBER 6
DIRECTION (NESWO) W
        .       .       .       .       .
        .       .       .       .       .
        .       .       .       .       .
        .       .       .       .       .
        Y       .       M       .       .



MOVE NUMBER 7
DIRECTION (NESWO) N
        .       .       .       .       .
        .       .       .       .       .
        .       .       .       .       .
        Y       M       .       .       .
        .       .       .       .       .



MOVE NUMBER 8
DIRECTION (NESWO) N
        .       .       .       .       .
        .       .       .       .       .
        Y       .       .       .       .
        M       .       .       .       .
        .       .       .       .       .



MOVE NUMBER 9
DIRECTION (NESWO) N
        .       .       .       .       .
        Y       .       .       .       .
        M       .       .       .       .
        .       .       .       .       .
        .       .       .       .       .



MOVE NUMBER 10
DIRECTION (NESWO) N
YOU SURVIVED!
PLAY AGAIN 
```

the 5x5 grid is still as hard as I remember it, but a bag of crisps as the prize for the highest score was a strong motivation to improve your playing!

### Challenges

The reason that a simple game like this can still be enoyable and interesting today even if it does not blink a single pixel on the screen, 
is that the book challenges you to modify the program and make it more interesting. 

The book contains the following challenges

Minor:
 1. Grid size 
 2. Turns to win 

Major: 
 1. Have more than one monster. 
 2. Chase a little monster while a big monster tries to get you. 
 3. Have the monster fall in quicksand. 
 4. Require food in order to maintain energy.

the _Minor_ ones are really easy and I will not bother, but I am really excited about the _Major_ ones as they have triggered some crazy ideas!

I am already planning them!


## Rating of this task

- Challenge: 3/10
- Satisfaction :10/10
- Funtime: 10/10
