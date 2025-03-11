> 518. Coin Change II
Medium

You are given an integer array coins representing coins of different denominations and an integer amount representing a total amount of money.

Return the number of combinations that make up that amount. If that amount of money cannot be made up by any combination of the coins, return 0.

You may assume that you have an *infinite number of each kind of coin.*  (hint: dealing with unbounded knapsack problem)

The answer is guaranteed to fit into a signed 32-bit integer.


```
Example 1:

Input: amount = 5, coins = [1,2,5]  
Output: 4  
Explanation: there are four ways to make up the amount:  
5=5  
5=2+2+1  
5=2+1+1+1  
5=1+1+1+1+1  

Example 2:

Input: amount = 3, coins = [2]  
Output: 0  
Explanation: the amount of 3 cannot be made up just  
with coins of 2.

Example 3:

Input: amount = 10, coins = [10]  
Output: 1
```

<table>
  <tr>
    <td rowspan="2">
      Coins
    </td>
    <td colspan="7">
      Amount
    </td>
  </tr>
  <tr>
  <td> See data tables below </td>
  </tr>
</table>

* Start filling in table

|   | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|
| 1 |   |   |   |   |   | 1 |
| 2 |   |   |   |   | X | 1 |    
| 5 |   |   |   |   | 0 | 1 |

* Trying to use the 2 coin here at X doesn't work because 1 - 2 = -1, outside the grid! hence use a 0.  Notice the 1's in the 0 amt column are the base case.

* Each cell takes the sum of values, coin value over right spaces and 1 below. Notice a 0 value means we can't make amount with given coin

|   | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|
| 1 |   |   |   |   |   | 1 |
| 2 |   |   |   | 1 | 0 | 1 |
| 5 | 1 | 0 | **0** | 0 | 0 | 1 |

* Table finished

|   | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|
| 1 | 4 | 3 | 2 | **2** | *1* | 1 |
| 2 | **1** | *1* | 0 | *1* | 0 | 1 |
| 5 | *1* | 0 | 0 | 0 | 0 | 1 |

* So for value **2** here, one value to the right is 1, and 1 value below is 1 so there are two combinations (e.g. 1+1 and 2)
* So for value **1** here, two values to the right is 0, and 1 value below is a 1 so we take the 1.  This corresponds to the coin combination of just 5
