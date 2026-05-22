# The Anatomy of a neareinf Run

This project maps out a methodology to obtain a "neareinf" score, one that is as close to the hard floating-point ceiling of *Balatro* as possible without going over the "naneinf" limit.

Special thanks go to u/jimbojonkler977 for the work done on the "977 naneinf" that helped to originally inspire this work, and u/ludic-sean for the original "naneinf" score run using Constellation.

---

## Finding Paths to "neareinf" Scores

The goal of this project is to find ways to score near, but not exceed, the score limit.  We will focus on the Plasma Deck and a standard formula that calculates a base mult based on the played hand, followed by a series of X1.5 triggers, to achieve a high score.

Balatro utilizes the [IEEE 754 double-precision floating-point format] (https://en.wikipedia.org/wiki/Double-precision_floating-point_format) to store score values. The largest normal number in this system is defined by the following limit $$I$$:

$$score_max = (2 - 2^{-52}) \times 2^{1023} \approx 1.7976931348623157 \times 10^{308}$$

To find the maximum allowed mult value when playing on the Plasma Deck, we calculate the square root of this value and multiply by two to account for the deck's chips-mult balancing algorithm:

$$plasmascore_max = 2 \times \sqrt{score_max} \approx 2.6816 \times 10^{154}$$

To get started, we want to find the upper bound for the number of triggers. When utilizing endgame strategies like Baron + King combinations and/or holding Steel Cards, every single trigger multiplies your current mult by X1.5.

We assume a base mult of 1 and solve for $$triggers_max$$

$$1 \times 1.5^{triggers_max} = {plasmascore_max}$$
$$\log_{1.5}(2 \times \sqrt{plasmascore_max}) \approx 876.9793$$

We round this value down to the nearest integer to find that the most X1.5 triggers you can have is 876.  Any more than this will exceed $$score_max$$.

We then solve to find the corresponding base mult for 876:

$${plasmascore_max} / (1.5^876) = 1.4875$$

Rounding this value down gives us a base mult target of 1.

Finally, we calculate the actual score for these two values:

$$S = 1 \times 1.5^876 = 8.1240e307$$

Here's a list of calculations from 876 down to 860.

| Triggers | Base Mult | Target Base Mult | Calculated Score |
| :--- | :--- | :--- | :--- |
| 876 | 1.4875 | 1 | 8.12504685697989e307 |
| 875 | 2.2312 | 2 | 1.44445277457420e308 |
| 874 | 3.3468 | 3 | 1.44445277457420e308 |
| 873 | 5.0202 | 5 | 1.78327503033852e308 |
| 872 | 7.5303 | 7 | 1.55343069309489e308 |
| 871 | 11.2954 | 11 | 1.70489899196809E+308 |
| 870 | 16.9431 | 16 | 1.60313734414630E+308 |
| 869 | 25.4146 | 25 | 1.73951534738097E+308 |
| 868 | 38.1219 | 38 | 1.78621167048400E+308 |
| 867 | 57.1829 | 57 | 1.78621167048400E+308 |
| 866 | 85.7743 | 85 | 1.76538139177824E+308 |
| 865 | 128.6615 | 128 | 1.77925466961290E+308 |
| 864 | 192.9923 | 192 | 1.77925466961290E+308 |
| 863 | 289.4884 | 289 | 1.79163205609494E+308 |
| 862 | 434.2326 | 434 | 1.79576738548958E+308 |
| 861 | 651.3490 | 651 | 1.79576738548958E+308 |
| 860 | 977.0235 | 977 | 1.79760683979717E+308 |

---

## Measuring Proximity: Logarithmic Headroom & Relative Precision

To measure exactly how close a score gets to $$score_max$$, we map two precise metrics that reveal the depth of our decimal alignment:

1. **Logarithmic Distance:** Measures the distance between the limit and the $$score$$ in orders of magnitude (powers of 10):
   $$distance = \log_{10}(score_max) - \log_{10}(score)$$
2. **Relative Precision (Significant Correct Digits):** Tracks how many leading decimal places the score shares with the limit:
   $$precision = -\log_{10}(distance)$$

Below is a table showing the first five entries of triggers and their respective base mult values, resulting calculated scores, and distance from $$L$$:

| Triggers | Base Mult | Target Base Mult | Calculated Score | Logarithmic Distance | Relative Precision (Digits) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **876** | 1.4875 | 1 | 8.12504685697989e307 | 0.34488968569115700 | 0.4623 |
| **875** | 2.2312 | 2 | 1.44445277457420e308 | 0.09501221247455760 | 1.0222 |
| **874** | 3.3468 | 3 | 1.44445277457420e308 | 0.09501221247455760 | 1.0222 |
| **873** | 5.0202 | 5 | 1.78327503033852e308 | 0.00349723135320801 | 2.4563 |
| **872** | 7.5303 | 7 | 1.55343069309489e308 | 0.06342367810810860 | 1.1977 |

A complete list is included in this project.  This list also includes the number of red seal steel and non-steel kings needed in hand to achieve each trigger amount, along with any Polychrome or other 1.5x scoring jokers to achieve specific trigger quantities.

---

## Finding Logical Options

We want to find logical pairs of triggers and base mult values so that the number of triggers and target base mult are achievable with our current situation. Ideally, a player is already on a standard naneinf run and has collected all the relevant vouchers and jokers (Baron, Mime, four copy jokers, ectoplasm, Antimatter, Retcon, etc.)

To find these values, the Balatro Hand Solver iterates through a subset of possible base mult values, looking for 1- to 5-card played hands that result in a specified calculated mult value.

As a basic (but impractical) example, a High Card hand of a mult card at level 973 will yield a mult value of 977.

More complicated, but more practical, examples include:
* Three of a Kind (level 39, base mult 79): Mult/Holo+Red, Glass/Poly+Red, Mult/Holo
* Four of a Kind (level 5, base mult 19): Glass/Holo+Red, Glass/None+Red, Mult/Poly+Red, Mult/None+Red
* Four of a Kind (level 25, base mult 79): Mult/Holo+Red, Glass/Poly+Red, Mult/Holo, 1 non-scoring card

In fact, there are over 98,000 different hand combinations to reach a target base mult level of 977.  The complete list of hands and hand levels to achieve base mult 977 is included in this project.

For other target base mult values, the balatro-hand-solver.html application iterates through every hand combination, hand type, and hand level to find all hands that calculate to the specific target base mult value.  The Hand Solver allows you to filter out specific cards that you don't have in your inventory (e.g. a Holographic Red Seal card) and filter out specific hands you don't want to attempt (e.g. High Card is impractical for most every scenario).

## Advanced Strategies

Base Mult 977 and 25040 can be reached using standard 4- and 5-card hands, leaving enough red seal steel and non-steel Kings in hand during a Burglar-Serpent run.  Anything beyond 25040 will likely require the use of additional scoring jokers through either natural negative jokers or Crimson Bean bug and additional ectoplasm Spectral cards.

In lieu of substantial hand leveling, larger base mult values (e.g. 3248853) also often require additional scoring jokers such as Constellation or Campfire.  To use these jokers, find the factor pairs of the target base mult value.  Use the scoring joker to account for one of the values, and use the hand solver to find a hand for the other value.

Example: 3248853 has factor pairs (3,1082951), (17,191109), and (51,63703).  This results in finding a hand that results in a base mult of 191109 while a Joker like Constellation or Campfire just needs to level up to 17.0.

## Disclosures
Portions of the codebase were generated using Google Gemini.
