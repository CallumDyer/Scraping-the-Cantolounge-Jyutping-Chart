## Scraping the Cantolounge Jyutping Chart

A tutorial on, and series of scripts for, scraping [Baggio Wong's](https://github.com/BaggioWongHK) cantolounge.com [Jyutping Chart](https://cantolounge.com/jyutping-chart/) and converting the individual audio files to consolidated drilling exercises of a desired length.

## Motivation

I was looking for Cantonese tone drills where I could listen and repeat the various sounds and tones in the language for around 15 minutes. The Cantolounge Jyutping Chart is a fantastic resource, but if one wants to use it daily to improve tones it is a bit tedious to navigate between each sound file and it is difficult to keep track of one's place over time. I therefore set out to scrape all the audio files and consolidate them into larger files, while also grabbing the Jyutping itself to follow along with.

It is worth noting that the audio files are all freely available [here](https://github.com/BaggioWongHK/jyutping-chart), however I believe that the raw audio files are not in the same order as their corresponding Jyutping. One way to preserve that order is to crawl the site.

## Tutorial

Start by navigating to https://baggiowonghk.github.io/jyutping-chart/.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/1_select_data.png)

Click on one of the sound links.
