Title: WaspStats virtual levels
Description: Explanation and logic behind the wasp-stats virtual levels.
Level: 0
Author: e40dee47-eb0c-4859-8434-44dd4d979673
Co-Authors: 

This is not exactly a tutorial, more an explanation on how virtual levels work in [wasp-stats](https://waspscripts.com/stats).

Take into consideration:
- You reach a 99 at 13 034 431 experience.

I considered using the orginal formulas from RuneScape, but the fact that they grow exponentially means after a while the levels would barely increase as you get higher and higher.

With this things in mind I decided I wanted have my virtual levels increase at a static value and that 99 should be the ceiling of the calculations.

With that said, the formula I came up with is:
```
level = floor(xp / (13034431 / 99))
```