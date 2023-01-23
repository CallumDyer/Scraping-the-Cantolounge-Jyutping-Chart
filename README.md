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

Paste the output into a text editor of your choice and save the file as 'chinese_from_JSONPath.txt'.

### Step 2: formatting the Chinese characters and inserting them into a list of urls

What we want is an array of Chinese characters, like the following:`"烏鴉"`. No doubt there are many ways to convert the data we have now in chinese_from_JSONPath.txt to such an array. The following is what I did, and requires Emacs (I will provide some links and instructions for those unfamiliar with it).

Create a new file titled 'script_1_formatting_chinese_raw_input.rb' (or alternatively use the script of the same name that is provided). Copy the contents of chinese_from_JSONPath.txt into it and assign the resulting array to the variable `chinese`. Note that for those following along with emacs, copying the characters in will result in emacs complaining about problematic characters. Simply press enter to `Select coding system (default utf-8)`. Emacs won't be able to fully render all the characters, but that does not matter for our purposes.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/18_chinese_variable.png)

Then add the following to the script (after the array of course):

```
chinese = chinese.reject { |c| c.empty? }

File.open("chinese_raw_text.txt", "w") {|f| f.write(chinese.join("\n"))}
```

I found two links helpful for writing this script, regarding [removing blank elements from an array](https://stackoverflow.com/questions/5878697/how-do-i-remove-blank-elements-from-an-array) and [how to create a file in Ruby](https://stackoverflow.com/questions/7911669/how-to-create-a-file-in-ruby).

The script removes blank elements from our `chinese` array and then writes out the elements to a new file 'chinese_raw_text.txt'. After running the script, chinese_raw_text.txt looks like this:

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/19_chinese_raw_text.png)

Next, we construct a list of urls using these characters. Copy the characters to a new script named 'script_2_crawling_cantolounge.rb'. 

Note that for emacs commands: `C-x h` or `C-x TAB` means hold down the ctrl key and while holding down the key, press x, immediately afterwards press the h or TAB keys respectively. `M-x` means hold down the alt key and while holding down the key, press x. If at any point you make a mistake, undo with `C-_` (to be clear, you will need to hold down ctrl and, since the _ character is reached with shift, also hold down shift and then press the _ key (which is also the key for - except without the shift).

Ensure the cursor is at the very start of the file by using the command `M-<` (to be clear, you will again need to hold down alt, hold down shift and press the < key (which is also the key for , except without the shift)).

Next, use `M-x` and then type in `replace-regexp` and press enter. You will be prompted to enter the regexp you want to match with, and then the expression you wish to replace. In our case, we want to replace the start of each line with the first part of our url. Type in `^` and press enter. Then type in `",` and press enter. The file should look like this:

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/20_replace-regexp_1.png)

Again, return to the start of the file with `M-<`. Use `M-x` and type in `replace-regexp` and press enter. This time, type in `$` and press enter. Type in `",` and press enter. Delete the comma from the final character. The file should look like this:

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/21_replace-regexp_2.png)

Indent the list to the right two spaces by first selecting all characters with `C-x h` and then use `C-x TAB` and press the right-arrow key twice. Add opening and closing square brackets to start and end of the list, respectively (the first row of characters may break its indentation as you add spaces above it, just re-indent it by pressing space twice). Assign the newly created array to the variable `characters`. The top and bottom of the file should look like the following:

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/22_characters_variable_1.png)
![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/23_characters_variable_2.png)

Add the following three lines to the start of the script:
```
require "open-uri"
require "fileutils"
require "CGI"
```
These are ruby gems you will need to install.

After the array, add the following code:
```
url_pattern = "https://baggiowonghk.github.io/jyutping-chart/audio/chinese/"

characters.each do |character|
  if character =~ /jaau1/
    puts "#{url_pattern}%E5%B7%A6%20jaau1.mp3"
    tempfile = URI.parse("#{url_pattern}%E5%B7%A6%20jaau1.mp3").open
    tempfile.close
    FileUtils.mv tempfile.path, "#{character}.mp3"
    next
  end
  elsif character =~ /long3/
    puts "#{url_pattern}long3%20%E9%AB%98.mp3"
    tempfile = URI.parse("#{url_pattern}long3%20%E9%AB%98.mp3").open
    tempfile.close
    FileUtils.mv tempfile.path, "#{character}.mp3"
    next
  end
  character_encoded = CGI.escape(character)
  puts "#{url_pattern}#{character_encoded}.mp3"
  tempfile = URI.parse("#{url_pattern}#{character_encoded}.mp3").open
  tempfile.close
  FileUtils.mv tempfile.path, "#{character}.mp3"
end
```

I will explain what this code is doing.

`url_pattern = "https://baggiowonghk.github.io/jyutping-chart/audio/chinese/"`

This is the first part of the urls we will be accessing. You will notice that this variable is called in this way: `"#{url_pattern}..."`. This is string interpolation. Ruby detects the `#{...}` syntax and inserts the value of the variable within into the string from which the syntax is being called. So `"#{url_pattern}%E5%B7%A6%20jaau1.mp3"` is effectively the same thing as `"https://baggiowonghk.github.io/jyutping-chart/audio/chinese/%E5%B7%A6%20jaau1.mp3"`.

```
characters.each do |character|
...
end
```

This is a loop. For each element of the array `characters`, we run the code within. In each iteration, the element is assigned to the variable in the pipes, `character`. We can access the element with this variable.

```
if character =~ /jaau1/
  puts "#{url_pattern}%E5%B7%A6%20jaau1.mp3"
  tempfile = URI.parse("#{url_pattern}%E5%B7%A6%20jaau1.mp3").open
  tempfile.close
  FileUtils.mv tempfile.path, "#{character}.mp3"
end
```

The code block within the if statement is triggered if `character` matches the regex pattern `jaau1`. We need to account for the urls which do not fit the normal pattern. Normally urls follow the pattern of `https://baggiowonghk.github.io/jyutping-chart/audio/chinese/烏鴉.mp3` or perhaps `https://baggiowonghk.github.io/jyutping-chart/audio/chinese/ngai1.mp3`. In the case of the element, `左 jaau1`, we need to encode the space, so the url will end up as `https://baggiowonghk.github.io/jyutping-chart/audio/chinese/左%20jaau1.mp3`. While we're at it, we can also encode the character 左. So instead of sticking the element `左 jaau1` at the end of our `url_pattern`,`https://baggiowonghk.github.io/jyutping-chart/audio/chinese/`, we instead stick the encoded character and space along with jaau1 and we end up accessing the url: `https://baggiowonghk.github.io/jyutping-chart/audio/chinese/%E5%B7%A6%20jaau1.mp3`.

```
tempfile = URI.parse("#{url_pattern}%E5%B7%A6%20jaau1.mp3").open
tempfile.close
FileUtils.mv tempfile.path, "#{character}.mp3"
```

I won't claim to know how exactly this code works. I came across it on a reddit comment by u/janko-m, found in this [post](https://www.reddit.com/r/ruby/comments/6x4ev4/how_to_download_an_mp3_file/). Nonetheless, what it ends up doing is downloading the mp3 file from the specified url and saving it with the name of the character.
