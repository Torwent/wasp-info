Title: Script stats
Description: Add stat tracking to your script!
Level: 0
Author: e40dee47-eb0c-4859-8434-44dd4d979673
Co-Authors: 

In this tutorial I'm going to go over how to add stats tracking to your scripts 📊

First of all, if you need more information that is not here, you should check the [`StatsPayload` and `APIClient` documentation](https://torwent.github.io/WaspLib/api.html).

With that said, let's begin!

To add stats to your script you need to have a script id assigned to your script, so if your script wasn't uploaded yet you can't. You can however add the logic in advance, SCRIPT_ID is added to the top of your script as soon as you first upload it.

When you are adding the script or editing later you will notice you have options to customize the stats limits.
This limits are for 5 minutes of the script running and any data sent to my API that doesn't fall within those limits will be rejected.
You should have some margin, but I would say 20% is fine.
Say your script earns 100k xp/h minimum and 200k xp/h maximum.

You will want the minimum xp to have a 20% margin which would mean 80k xp/h minimum and you would need to convert that to 5mins:
```
 6666 = 80000/60*5
```
Let's set it to 6500 for some extra margin and to have a nice round number but it's up to you really.

As for the maximum xp, 20% margin would be 240k xp/h max and converted to 5 mins would be:
```
20000 = 240000/60*5
```
We can set the max xp to 20000.
 
Do the same for GP and save your settings. With that the configuration on the website is done.

Now onto the script!

Depending on what you use of Wasplib, and what you want to track you might not even need anything at all.
For example, if you use WaspLib's `TBaseScript` types, `TBaseScript.Init()` will automatically set things up for you and `XPBar` methods like `EarnedXP()` and `WaitXP()` track the xp for you and finally `TBaseScript.PrintReport()` will submit your stats.

So if you only track xp and use those, you are done.

Now if your don't use `TBaseScript.Init()`, you will need to add this to the beginning of your script, be it on a startup method or whatever you use:
```pascal
StatsPayload.Setup({$macro SCRIPT_ID});
APIClient.TimeStamp := GetTickCount();
```
Again, this is only required if you don't use `TBaseScript.Init()` or any of it's ""descendants"".

Now, for the `StatsPayload`! This is the data that will be sent to my API server.
The way it works is that your should only update it with values between requests.

Imagine that you gain 1 gp every 1 second and you are sending the payload every 30 seconds.
30 seconds pass and you earned 30gp.

You could update StatsPayload 30 times adding 1 gp each second or adding 30 gp once every 30 seconds.

`StatsPayload` will track the amount you added between requests and will send the total of it once the request is sent and `StatsPayload` values are reset back to 0.

XP is exactly the same. But more on this soon.

You can update `StatsPayload` with the following method:
```pascal
StatsPayload.Update(xp, gp, runtime);
```
The `runtime` value should always be 0 unless you do your own POST implementation which won't be covered.

For `xp`, you can either implement your own tracking with the help of `XPBar.Read()` or whatever you like, but the easiest way is to simply call WaspLib's `XPBar.EarnedXP()` before you send the data to my API Client and that will track the xp by itself.
Realistically, calling it or `XPBar.WaitXP()` anywhere on your script is enough, but using it right before sending the data will ensure that the most up to date data is sent instead of maybe slightly outdated data.
Whatever you see fit will do.
If you do choose to do your own implementation it will work exactly as the `gp` explanation below.

For `gp` it will always be up to you how you track it. Just keep in mind the example I gave before and that it should be reset between requests and not the total your script has earned.

Imagine your max gp limit is set to 5k every 5 minutes.
If you are sending the total gp your script as earned everytime, once your script has earned over 50k gp, all your requests will be rejected from that point on.

Personally I like to update every time I know I earned or lost GP.
E.G. my astral rcer, everytime it teleports it does:
```pascal
StatsPayload.Update(0, -Self.TeleportPrice, 0);
```
And it has `TeleportPrice` setup on my `Init()` method.
Then everytime it makes runes:
```pascal
    invCount -= Inventory.Count();
    if not hasAstrals then
      invCount += 1;

    astralCount += Inventory.CountItemStack(Self.Rune.Item) - Self.TotalAstrals;
    profit := Self.AstralPrice * astralCount - Self.EssPrice * invCount;
    StatsPayload.Update(0, profit, 0);
    Self.TotalProfit += profit;
    Self.TotalActions += invCount;
```
Without going into details on all this, it basically is just counting the profit of that one inventory and updating `StatsClient` with `StatsPayload.Update(0, profit, 0);`.

Now finally, sending the data to the API server!
If you use `TBaseScript.PrintReport()`, you don't need anything. It does it for you.

If you don't, you will need to add to add the following to your main loop:
```pascal
if not APIClient.IsSetup or APIClient.Timer.IsFinished() then
    APIClient.SubmitStats(APIClient.GetUUID());
```
You do need the if statement, otherwise you might spam my server and you will get rate limited. You are allowed to do 3 requests per 4 seconds otherwise you get rate limited and your requests rejected.
A good place to add this in my opinion is in your report printer if you have one.

Lastly, if you were to update the `StatsPayload` only once per request, you would want to do it right above this:
```pascal
XPBar.EarnedXP(); //ensure StatsPayload.Experience is fully up to date.
StatsPayload.Update(0, GPEarnesSinceLastRequest, 0); //xp is 0 because XPBar.EarnedXP() already updated it.
if not APIClient.IsSetup or APIClient.Timer.IsFinished() then
    APIClient.SubmitStats(APIClient.GetUUID());
```

And with that you are done, you have stats tracking added to your script😁