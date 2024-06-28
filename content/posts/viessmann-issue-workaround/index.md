---

title: "A Smart Home Challenge: Resolving a Viessmann Heat Pump Problem"

date: "2024-06-28"

description: How I Found and Fixed an Issue with My Viessmann Heat Pump

---

Today, I want to share a personal story about how I discovered and successfully resolved an issue with my Viessmann heat pump. A smart home is full of little technical marvels, but occasionally, challenges arise. This experience has taught me the importance of collecting and analyzing data to effectively solve technical problems.

## My Viessmann Heat Pump and the First Winter
We have had our heat pump in operation since moving in 2021. During the first winter, I noticed an issue: the temperature of the hot water tank rapidly dropped by about 1 degree per hour as soon as the heating function was turned on. At that time, I lacked the knowledge and technical means to accurately identify the problem.

## Diagnosis With Home Assistant
Two years later, I decided to integrate the heat pump into Home Assistant using vcontrold. This integration allowed me to graphically display exact temperatures. It quickly became evident that the temperature of the hot water tank began to drop precisely when the heating function was activated.

## First Solution Attempt
After diagnosing the problem, I contacted Viessmann’s customer service, who replaced the diverter valve. This replacement reduced the heat loss from about 1 degree per hour to approximately 0.5 degrees per hour but did not completely solve the issue.

## The Real Cause and Final Solution
Despite the replaced diverter valve, there was still a minor circulation issue. This residual circulation caused water from the heating operation to flow through the heat exchanger in the hot water tank, cooling it down as it lost heat to the underfloor heating.

To finally address this, I installed an actuator on the ball valve of the return line behind the hot water tank. Using a Shelly device and a Home Assistant automation, the ball valve now automatically closes when hot water is not being produced. This effectively prevents unwanted circulation. Since the original motor from Heimeier was too expensive, I purchased a cheaper motor and 3D-printed an adapter.

As a comparison, these three charts, all covering a range of 24 hours:

- before changing the diverter valve:

{{< figure src="before-diverter.JPEG" title="" >}}

- after changing the diverter valve:

{{< figure src="after-diverter.JPEG" title="" >}}

- after installing an actuator on the ball valve:

{{< figure src="ball-valve-closed.png" title="" >}}

## Result
Thanks to this measure, the heat pump now loses only about 0.2 degrees per hour and functions flawlessly. A significant success after a long search!

## Lessons Learned
The most important takeaway from this experience is the value of collecting and analyzing data from the heat pump. It was particularly helpful to not rely on data from the Viessmann Cloud, as those data are updated infrequently. By integrating directly into Home Assistant, I had access to much more precise and up-to-date information.

## Conclusion
Technical challenges in a smart home can be daunting, but they also present an opportunity to learn more about the systems around us. With the right tools and a bit of patience, many issues can be resolved. I hope my experiences and tips will help you tackle your own smart home problems!