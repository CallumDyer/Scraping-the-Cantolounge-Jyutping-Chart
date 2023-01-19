# Scraping the Cantolounge Jyutping Chart

A tutorial on, and series of scripts for, scraping [Baggio Wong's](https://github.com/BaggioWongHK) cantolounge.com [Jyutping Chart](https://cantolounge.com/jyutping-chart/) and converting the individual audio files to consolidated drilling exercises of a desired length.

## Motivation

I was looking for Cantonese tone drills where I could listen and repeat the various sounds and tones in the language for around 15 minutes. The Cantolounge Jyutping Chart is a fantastic resource, but if one wants to use it daily to improve tones it is a bit tedious to navigate between each sound file and it is difficult to keep track of one's place over time. I therefore set out to scrape all the audio files and consolidate them into larger files, while also grabbing the Jyutping itself to follow along with.

It is worth noting that the audio files are all freely available [here](https://github.com/BaggioWongHK/jyutping-chart), however I believe that the raw audio files are not in the same order as their corresponding Jyutping. One way to preserve that order is to crawl the site.

## Tutorial

### Step 1: grabbing the Chinese characters

Navigate to https://baggiowonghk.github.io/jyutping-chart/ and click on one of the sound links.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/1_select_data.png)

Copy the Jyutping that appears.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/2_copy_jyutping_text.png)

Open Chrome DevTools with ctrl+shift+c or by right clicking anywhere on the page, and selecting the last option of 'Inspect'.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/3_get_to_chrome_dev_tools.png)

From Chrome DevTools, select the 'Network' tab.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/4_network_tab.png)

Refresh the page with ctrl+r or by selecting the refresh button in your browser.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/5_refresh_network_tab.png)

Enter the text we copied earlier into the search bar and hit enter (if the search bar isn't visible, use ctrl+f to summon it).

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/6_enter_text_network_tab_search.png)

Select the result that appears below.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/7_select_network_tab_search_result.png)

Select the resulting window and use ctrl+a to select all of the window's text, then copy.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/8_copy_resulting_json.png)

Find somewhere to paste the text and delete the parts that are not valid JSON. I used jsonlint.com. The first part to delete is at the start. Delete everything up to `var samplewords = `.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/9_delete_javascript.png)

The second part to delete is at the end. Delete everything after the second last semicolon of the document.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/10_delete_javascript_2.png)

The final part to delete is the comma on the now second last curly bracket of the document.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/11_delete_comma.png)

If you're following along on jsonlint.com, click on the 'Validate JSON' button to make sure you got everything.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/12_click_validate.png)

There will be an error that says 'Error: Duplicate key 'deu''. That's fine.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/13_disregard_error.png)

Select the text and use ctrl+a and copy the now valid (for our purposes) JSON.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/14_copy_JSON.png)

Navigate to [JSONPath.com](https://jsonpath.com/) and paste the JSON into 'Inputs'.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/15_paste_JSON.png)

Type in the JSONPath: `*.*.chinese` into the JSONPath input bar above.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/16_Chinese_JSONPath.png)

Select the resulting 'Evaluation Results' and copy the output with ctrl-A. Keep this same tab open as we'll need it later for grabbing the Jyutping.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/17_copy_Chinese.png)

Paste the output into a text editor of your choice and save the file as 'chinese.txt'.

### Step 2: formatting the Chinese characters and inserting them into a list of urls
