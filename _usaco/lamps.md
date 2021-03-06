---
layout: default
title: "Party Lamps"
date: 2020-07-01 12:00:00 -0700
author: Eddy Yu
category: usaco
tags: [usaco]
order: 2.2.4
---

##### Note: description is copied from [USACO](http://www.usaco.org/){:target="_blank" rel="noopener"} training website and converted to markdown

### Description:
To brighten up the gala dinner of the IOI'98 we have a set of N (10 <= N <= 100)
colored lamps numbered from 1 to N.

The lamps are connected to four buttons:

* Button 1: When this button is pressed, all the lamps change their state: those 
  that are ON are turned OFF and those that are OFF are turned ON.
* Button 2: Changes the state of all the odd numbered lamps.
* Button 3: Changes the state of all the even numbered lamps.
* Button 4: Changes the state of the lamps whose number is of the form 3xK+1 
  (with K>=0), i.e., 1,4,7,...

A counter C records the total number of button presses.

When the party starts, all the lamps are ON and the counter C is set to zero.

You are given the value of counter C (0 <= C <= 10000) and the final state of 
some of the lamps after some operations have been executed. Write a program to 
determine all the possible final configurations of the N lamps that are 
consistent with the given information, without repetitions.

##### PROGRAM NAME: lamps

##### INPUT FORMAT

Line 1: | N
--------|---------------------
Line 2: | Final value of C
Line 3: | Some lamp numbers ON in the final configuration, separated by one space and terminated by the integer -1.
Line 4: | Some lamp numbers OFF in the final configuration, separated by one space and terminated by the integer -1.

##### SAMPLE INPUT (file lamps.in)
```
10
1
-1
7 -1
```
In this case, there are 10 lamps and only one button has been pressed. Lamp 7 
is OFF in the final configuration.

##### OUTPUT FORMAT
Lines with all the possible final configurations (without repetitions) of all 
the lamps. Each line has N characters, where the first character represents the 
state of lamp 1 and the last character represents the state of lamp N. A 0 (zero)
stands for a lamp that is OFF, and a 1 (one) stands for a lamp that is ON. The 
lines must be ordered from least to largest (as binary numbers).

If there are no possible configurations, output a single line with the single 
word 'IMPOSSIBLE'.

##### SAMPLE OUTPUT (file lamps.out)
```
0000000000
0101010101
0110110110
```
In this case, there are three possible final configurations:
* All lamps are OFF
* Lamps 1, 3, 5, 7, 9 are OFF and lamps 2, 4, 6, 8, 10 are ON.
* Lamps 1, 4, 7, 10 are OFF and lamps 2, 3, 5, 6, 8, 9 are ON.

### Analysis:
There are 4^C (e.g. 4^10000) possible combinations, which is way too big! So, 
that means, there must be some optimization in order to solve this problem.

Some observations:
* The order of the button click does not matter (e.g. clicking button 1 and 
  then button 2 is the same as clicking button 2 and then button 1).
* Clicking the same button more than once will just cancel out that button 
  click. 

Hence, you can reduce the total combinations by ignoring the button click 
order and by only having at most 4 clicks per combo.
    
### Solution:
```java
public class lamps {

    public static void main(String[] args) throws IOException {
        try (final BufferedReader f = new BufferedReader(new FileReader("lamps.in"));
             final PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter("lamps.out")))) {

            // number of lamps
            final int N = Integer.parseInt(f.readLine());

            // total number of button presses
            final int C = Integer.parseInt(f.readLine());

            // final lamp configuration
            final int[] finalLampConfiguration = new int[N]; // 0 = off, 1 = on, 3 = off or on
            Arrays.fill(finalLampConfiguration, 3);
            final Scanner scanner = new Scanner(f);
            int lampNumber;
            while ((lampNumber = scanner.nextInt()) != -1) {
                finalLampConfiguration[lampNumber - 1] = 1; // ON
            }
            while ((lampNumber = scanner.nextInt()) != -1) {
                finalLampConfiguration[lampNumber - 1] = 0; // OFF
            }

            // generate all possible click combinations
            final int maxClicks = Math.min(C, 4); // max of 4 clicks
            final int[] combo = new int[maxClicks];
            final List<int[]> combos = new ArrayList<>();
            generateClickCombos(combo, 0, maxClicks, combos);

            // compute the results of the combos
            final List<int[]> results = new ArrayList<>();
            computeResults(N, combos, results);

            // find results which are consistent with the final lamp configuration
            final Set<String> validResults = new HashSet<>();
            for (int[] result : results) {
                boolean isMatch = true;
                for (int i = 0; i < finalLampConfiguration.length; i++) {
                    if (finalLampConfiguration[i] == 3) {
                        continue; // any state is allowed; check next lamp
                    }
                    if (finalLampConfiguration[i] != result[i]) {
                        // desired state is not matched
                        isMatch = false;
                        break;
                    }
                    // else - desired state is matched
                }
                if (isMatch) {
                    validResults.add(Arrays.toString(result).replaceAll("[^0-9]", ""));
                }
            }

            // output result
            if (validResults.size() == 0) {
                out.println("IMPOSSIBLE");
            } else {
                // sort the results
                String[] validResultsArray = validResults.toArray(new String[0]);
                Arrays.sort(validResultsArray);
                for (String resultString : validResultsArray) {
                    out.println(resultString);
                }
            }
        }
    }

    private static void generateClickCombos(int[] combo, int currentClickCount,
                                            int maxClicks, List<int[]> combos) {
        // combo generation is complete
        if (currentClickCount == maxClicks) {
            combos.add(Arrays.copyOf(combo, combo.length));
            return;
        }

        // else find next button (button1 to button4) for next click in combo
        for (int i = 1; i <= 4; i++) {
            combo[currentClickCount] = i;
            generateClickCombos(combo, (currentClickCount + 1), maxClicks, combos);
        }
    }

    private static void computeResults(int N, List<int[]> combos, List<int[]> results) {
        // iterate through the combos and compute the resulting configuration of the lamps
        for (int[] combo : combos) {
            final int[] lamps = new int[N]; // 0 = off, 1 = on
            Arrays.fill(lamps, 1); // all lamps start in ON state
            // check which button is pressed for each click
            for (int button : combo) {
                if (button == 1) {
                    // button1 : all lamps flip state
                    for (int y = 0; y < lamps.length; y++) {
                        lamps[y] = (lamps[y] + 1) % 2;
                    }
                } else if (button == 2) {
                    // button2 : all odd lamps flip state
                    for (int y = 0; y < lamps.length; y = y + 2) {
                        lamps[y] = (lamps[y] + 1) % 2;
                    }
                } else if (button == 3) {
                    // button3: all even lamps flip state
                    for (int y = 1; y < lamps.length; y = y + 2) {
                        lamps[y] = (lamps[y] + 1) % 2;
                    }
                } else if (button == 4) {
                    // button4: all 3xK+1 lamps flip state, where K=click
                    for (int k = 0; k < lamps.length; k++) {
                        int j = (3 * k) + 1;
                        // make sure we don't exceed array bounds
                        if (j - 1 < lamps.length) {
                            lamps[j - 1] = (lamps[j - 1] + 1) % 2;
                        }
                    }
                }
            }
            results.add(Arrays.copyOf(lamps, lamps.length));
        }
    }
}
``` 
Link To: [Java Source Code](https://github.com/eddycyu/usaco/blob/master/src/barn1.java){:target="_blank" rel="noopener"}

### Links:
* [Find The Nth Fibonacci Number In The Sequence Using Recursion](/blog/find-nth-fibonacci-number-recursion)
