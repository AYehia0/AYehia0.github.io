---
title: "Use you home modem when electricity is down !"
date: 2023-11-27T18:21:53+02:00
lastmod: 2023-11-27T18:21:53+02:00
draft: true
toc:
  enable: true
  auto: false
share:
  enable: true

authors: []
description: ""

tags: [diy-projects]
categories: [electronics]
series: []

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: "featured-image.jpg"
featuredImagePreview: "featured-image.jpg"
---
In the palce I live in, electricity goes off everyday for 2 hours and I really need the internet connectoin that time, because you know it's 2023, Internet is a like water, no one can live without it.
<!--more-->

## Getting Started
Before getting started with the project, make sure you have the required tools and materials:
- Battaries (for my case I used a power bank).
- Voltage Boost converter (DC-DC)
- USB cable.
- The other part of the modem connection.
- Avometer to check voltages.
- Solder Flux and soldering iron tool.

{{< image src="power_bank_usb_modem_cable.jpg" caption="Power bank, USB cable, and the modem cable." >}}
{{< image src="voltage_regulator.jpg" caption="Voltage Regulator (Step Up)" >}}

## Check for faulty parts
Before soldering, make sure the cables are working properly be checking the input/output voltage using the Avometer.

### Power Bank output through the USB cable.
Usually the power bank outputs 5V, which will be later steped up to 12V (input voltage of the home modem).
{{< image src="power_bank_output.jpg" caption="Power bank output through the USB cable." >}}

### Final output for the regulator
1. Check out your home modem's output (you can find it in the back of your modem)
2. Adjust you the booster to match the desired output by turing around the potentiometer.
{{< image src="final_output.jpg" caption="Power bank output through the USB cable." >}}

