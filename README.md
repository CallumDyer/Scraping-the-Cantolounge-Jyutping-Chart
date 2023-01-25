# Scraping the Cantolounge Jyutping Chart

A tutorial on, and series of scripts for, scraping [Baggio Wong's](https://github.com/BaggioWongHK) cantolounge.com [Jyutping Chart](https://cantolounge.com/jyutping-chart/) and converting the individual audio files to consolidated drilling exercises of a desired length.

The consolidated drilling exercises are available [here](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/tree/main/Scripts%20and%20files/Ready%20audio%20files%20and%20word%20docs) for those who may want them without following the tutorial.

## Motivation

I was looking for Cantonese tone drills where I could listen and repeat the various sounds and tones in the language for around 15 minutes. The Cantolounge Jyutping Chart is a fantastic resource, but if one wants to use it daily to improve one's tones it is a bit tedious to navigate between each sound file and it is difficult to keep track of one's place over time. I therefore set out to scrape all the audio files and consolidate them into larger files, while also grabbing the Jyutping itself to follow along with.

It is worth noting that the audio files are all freely available [here](https://github.com/BaggioWongHK/jyutping-chart), however I believe that the raw audio files are not in the same order as their corresponding Jyutping. One way to preserve that order is to crawl the site.

## Summary of what I did

1. Grabbed JSON from the site and used it to get a list of characters
2. Formatted the list of characters
3. Included the list as an array in a script which systematically downloaded each audio file (the urls for which made reference to each character)
4. Combined the audio files into one big file
5. Split the big file into 5 seperate files
6. Prepared a doc with both the characters and Jyutping to follow along with while drilling

## Tutorial

### Step 0: prerequisites

Ideally you should have some familiarity with programming, particularly Ruby, and you should know the basics of navigating the command line. That being said, I think even someone with no programming experience should be able to follow along (albeit, perhaps, with significant effort and a bunch of googling).

### Step 1: grabbing the Chinese characters

Navigate to https://baggiowonghk.github.io/jyutping-chart/ and click on one of the sound links.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/1_select_data.png)

Copy the Jyutping that appears.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/2_copy_jyutping_text.png)

Open Chrome DevTools with `ctrl+shift+c` or by right clicking anywhere on the page, and selecting the last option of 'Inspect'.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/3_get_to_chrome_dev_tools.png)

From Chrome DevTools, select the 'Network' tab.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/4_network_tab.png)

Refresh the page with `ctrl+r` or by selecting the refresh button in your browser.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/5_refresh_network_tab.png)

Enter the text we copied earlier into the search bar and hit enter (if the search bar isn't visible, use `ctrl+f` to summon it).

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/6_enter_text_network_tab_search.png)

Select the result that appears below.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/7_select_network_tab_search_result.png)

Select the resulting window and use `ctrl+a` to select all of the window's text, then copy.

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

Select the text and use `ctrl+a` and copy the now valid (for our purposes) JSON.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/14_copy_JSON.png)

Navigate to [JSONPath.com](https://jsonpath.com/) and paste the JSON into 'Inputs'.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/15_paste_JSON.png)

Type in the JSONPath: `*.*.chinese` into the JSONPath input bar above.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/16_Chinese_JSONPath.png)

Select the resulting 'Evaluation Results' and copy the output with `ctrl+a`. Keep this same tab open as we'll need it later for grabbing the Jyutping.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/17_copy_Chinese.png)

Paste the output into a text editor of your choice and save the file as `chinese_from_JSONPath.txt`.

### Step 2: formatting the Chinese characters

What we want is an array of Chinese characters, like the following:`"烏鴉"`. No doubt there are many ways to convert the data we have now in chinese_from_JSONPath.txt to such an array. The following is what I did, and requires Emacs (I will provide some links and instructions for those unfamiliar with it).

Create a new file titled `script_1_formatting_chinese_raw_input.rb` (or alternatively use the script of the same name that is provided). Copy the contents of `chinese_from_JSONPath.txt` into it and assign the resulting array to the variable `chinese`. Note that for those following along with emacs, copying the characters in will result in emacs complaining about problematic characters. Simply press enter to `Select coding system (default utf-8)`. Emacs won't be able to fully render all the characters, but that does not matter for our purposes.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/18_chinese_variable.png)

Then add the following to the script (after the array of course):

```
chinese = chinese.reject { |c| c.empty? }

File.open("chinese_raw_text.txt", "w") {|f| f.write(chinese.join("\n"))}
```

I found two links helpful for writing this script, regarding [removing blank elements from an array](https://stackoverflow.com/questions/5878697/how-do-i-remove-blank-elements-from-an-array) and [how to create a file in Ruby](https://stackoverflow.com/questions/7911669/how-to-create-a-file-in-ruby).

The script removes blank elements from our `chinese` array and then writes out the elements to a new file `chinese_raw_text.txt`. After running the script, `chinese_raw_text.txt` looks like this:

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/19_chinese_raw_text.png)

Next, we construct a list of urls using these characters. Copy the characters to a new script named `script_2_crawling_cantolounge.rb`. 

Note that for emacs commands: `C-x h` or `C-x TAB` means hold down the ctrl key and while holding down the key, press x, immediately afterwards press the h or TAB keys respectively. `M-x` means hold down the alt key and while holding down the key, press x. If at any point you make a mistake, undo with `C-_` (to be clear, you will need to hold down ctrl and, since the _ character is reached with shift, also hold down shift and then press the _ key (which is also the key for - except without the shift)).

Ensure the cursor is at the very start of the file by using the command `M-<` (to be clear, you will again need to hold down alt, hold down shift and press the < key (which is also the key for , except without the shift)).

Next, use `M-x` and then type in `replace-regexp` and press enter. You will be prompted to enter the regexp you want to match with, and then the expression you wish to replace. In our case, we want to replace the start of each line with the first part of our url. Type in `^` and press enter. Then type in `"` and press enter. The file should look like this:

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/20_replace-regexp_1.png)

Again, return to the start of the file with `M-<`. Use `M-x` and type in `replace-regexp` and press enter. This time, type in `$` and press enter. Type in `",` and press enter. Delete the comma from the final character. The file should look like this:

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/21_replace-regexp_2.png)

Indent the list to the right two spaces by first selecting all characters with `C-x h` and then use `C-x TAB` and press the right-arrow key twice. Add opening and closing square brackets to start and end of the list, respectively (the first row of characters may break its indentation as you add spaces above it, just re-indent it by pressing space twice). Assign the newly created array to the variable `characters`. The top and bottom of the file should look like the following:

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/22_characters_variable_1.png)
![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/23_characters_variable_2.png)

### Step 3: finishing the download script

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

characters.each_with_index do |character, index|
  if character =~ /jaau1/
    puts "#{url_pattern}%E5%B7%A6%20jaau1.mp3"
    tempfile = URI.parse("#{url_pattern}%E5%B7%A6%20jaau1.mp3").open
    tempfile.close
    FileUtils.mv tempfile.path, "#{index}.#{character}.mp3"
    next
  elsif character =~ /long3/
    puts "#{url_pattern}long3%20%E9%AB%98.mp3"
    tempfile = URI.parse("#{url_pattern}long3%20%E9%AB%98.mp3").open
    tempfile.close
    FileUtils.mv tempfile.path, "#{index}.#{character}.mp3"
    next
  end
  character_encoded = CGI.escape(character)
  puts "#{url_pattern}#{character_encoded}.mp3"
  tempfile = URI.parse("#{url_pattern}#{character_encoded}.mp3").open
  tempfile.close
  FileUtils.mv tempfile.path, "#{index}.#{character}.mp3"
end
```

I will explain what this code is doing.

`url_pattern = "https://baggiowonghk.github.io/jyutping-chart/audio/chinese/"`

This is the first part of the urls we will be accessing. You will notice that this variable is called in this way: `"#{url_pattern}..."`. This is string interpolation. Ruby detects the `#{...}` syntax and inserts the value of the variable within into the string from which the syntax is being called. So `"#{url_pattern}%E5%B7%A6%20jaau1.mp3"` is effectively the same thing as `"https://baggiowonghk.github.io/jyutping-chart/audio/chinese/%E5%B7%A6%20jaau1.mp3"`.

```
characters.each_with_index do |character, index|
...
end
```

This is a loop. For each element of the array `characters`, we run the code within. In each iteration, the element is assigned to the variable in the pipes, `character`. We can access the element with this variable. We are also able to access the index for each iteration with `index`.

```
if character =~ /jaau1/
  puts "#{url_pattern}%E5%B7%A6%20jaau1.mp3"
  tempfile = URI.parse("#{url_pattern}%E5%B7%A6%20jaau1.mp3").open
  tempfile.close
  FileUtils.mv tempfile.path, "#{index}.#{character}.mp3"
  next
end
```

The code block within the if statement is triggered if `character` matches the regex pattern `jaau1`. We need to account for the urls which do not fit the normal pattern. Normally urls follow the pattern of `https://baggiowonghk.github.io/jyutping-chart/audio/chinese/烏鴉.mp3` or perhaps `https://baggiowonghk.github.io/jyutping-chart/audio/chinese/ngai1.mp3`. In the case of the element, `左 jaau1`, we need to encode the space, so the url will end up as `https://baggiowonghk.github.io/jyutping-chart/audio/chinese/左%20jaau1.mp3`. While we're at it, we can also encode the character 左. So instead of sticking the element `左 jaau1` at the end of our `url_pattern`,`https://baggiowonghk.github.io/jyutping-chart/audio/chinese/`, we instead stick the encoded character and space along with jaau1 and we end up accessing the url: `https://baggiowonghk.github.io/jyutping-chart/audio/chinese/%E5%B7%A6%20jaau1.mp3`. The index, `#{index}`, allows us to keep our downloaded files in order.

```
tempfile = URI.parse("#{url_pattern}%E5%B7%A6%20jaau1.mp3").open
tempfile.close
FileUtils.mv tempfile.path, "#{index}.#{character}.mp3"
```

I won't claim to know how exactly this code works. I came across it on a reddit comment by u/janko-m, found in this [post](https://www.reddit.com/r/ruby/comments/6x4ev4/how_to_download_an_mp3_file/). Nonetheless, what it ends up doing is downloading the mp3 file from the specified url and saving it with the name of the character.

`next` simply skips to the next element in the array.

```
elsif character =~ /long3/
  puts "#{url_pattern}long3%20%E9%AB%98.mp3"
  tempfile = URI.parse("#{url_pattern}long3%20%E9%AB%98.mp3").open
  tempfile.close
  FileUtils.mv tempfile.path, "#{index}.#{character}.mp3"
  next
end
```
The `elsif` statement does the same as the `if` statement except for `long3` instead of `jaau1`.

```
character_encoded = CGI.escape(character)
puts "#{url_pattern}#{character_encoded}.mp3"
tempfile = URI.parse("#{url_pattern}#{character_encoded}.mp3").open
tempfile.close
FileUtils.mv tempfile.path, "#{index}.#{character}.mp3"
```

For all other characters, other than those including `jaau1` and `long3`, we encode the characters with `character_encoded = CGI.escape(character)` so that `URI.parse("#{url_pattern}#{character_encoded}.mp3").open` works properly.

Before running the script, there are two characters we need to manually delete from our array, since there are no urls for them. They are 吓 and 𥄫. Delete the rows with these characters, or else the script will break when it reaches them (the urls will return a 404 error).

Now run the script by navigating to the directory containing the script (note that in this directory will be these hundreds of mp3 files, so you may want to run it in an empty folder) using `ruby script_2_crawling_cantolounge.rb`. Once the script finishes running, you should have 1632 mp3 files (including the first file, which has the zeroth index).

### Step 4: combining the downloaded mp3s

Use FFmpeg to combine the files. If you have Windows Subsystem for Linux (WSL), you can install easily install it with [these commands](https://gist.github.com/ScottJWalter/eab4f534fa2fc9eb51278768fd229d70).

We will construct a .txt file for the input for FFmpeg. Navigate to the directory on the command line, and use the https://unix.stackexchange.com/questions/33909/list-files-sorted-numerically `ls -1v`, then copy all the file names in order and paste them in Emacs. Copying may be a bit tedious, I wasn't able to find a way to copy all the results with the keyboard using WSL. Use right-click to copy on the terminal.

Create a new .txt file called `input_for_FFmpeg.txt` in the directory where you have your .mp3 files. Then, use the same method as before to fill out the rest of the .txt. Use `M-x <` to navigate to the start of the file, use `M-x`, type in `replace-regexp`, type in `^` and finally type in `file /home/callumdyer/Cantonese/`, except with whatever the path is to the directory where you have your .mp3 files (rather than mine). Your file should look something like this:

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/24_input_for_ffmpeg.png)

You will need to delete the space from the jaau1 and long3 and long .mp3 file names, so that there's no space between the character and the Jyutping (142.左jaau1.mp3 vs 142.左 jaau1.mp3). Do this as well within the `input_for_FFmpeg.txt` file, otherwise FFmpeg will return an error when it gets to these two files.

Using Ubuntu or WSL Ubuntu, after navigating to the directory with the .txt file and the mp3 files, use the following command: `sudo ffmpeg -f concat -safe 0 -i input_for_FFmpeg.txt tone_drills.mp3`

After FFmpeg finishes, you should have a 1 hour, 21 minutes and 59 seconds long .mp3 file that contains all the individual .mp3 files.

### Step 5: splitting into different files again

First, decide how many files you want, or how long you want each file to be. If you want 5 files (this is what I chose), each will be 16 minutes 23.8 seconds. 4 will be about 20 minutes. 6 will be about 13 minutes. You can use this [calculator](https://www.calculatorsoup.com/calculators/time/time-calculator.php) for making the calculations.

We will use Audacity to split the files. Open `tone_drills.mp3` in Audacity. Ensure the audio is selected by pressing `ctrl+a`. At the bottom, you'll see two bars with times in them, labelled above as 'Start and End of Selection'. For the 'end', the bar on the right, the time for the end of the file should be present, about 1:21:59. Next, we'll use our calculated time of 16 minutes and 23.8 seconds, rounded up to 16 minutes and 24 seconds (the last file will be slightly shorter). We want our file to be 16:24, so enter that as the 'start', the bar on the left. Audacity should look like this:

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/25_audacity_1.png)

Cut this selection using either the drop-down menu from right-clicking, or from using `ctrl+x`, so that only the first 16:24 are left. Like this:

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/26_audacity_2.png)

Now select 'File' in the top left, then 'Export' and choose 'Export as MP3' or 'Export as WAV'.

Now open a new file and paste in the audio we cut. This should equal 1:05:35 (or so), since this is 1:21:59 minus 16:24. Repeat the same process using `ctrl+a`, entering 16:24 into the start of selection and then cutting and exporting the remaining 16:24 clip.

Repeat until you have split off all the desired files.

### Step 6: creating accompanying docs

Grab all the characters from `chinese_raw_text.txt` which was the output of `script_1_formatting_chinese_raw_input.rb`. Paste these into a spreadsheet program of your choice. 

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/27_spreadsheet_1.png)

For convenience sake, go to the bottom of the characters using `ctrl+down arrow`, then add a period to the four cells next to the last character in columns B, C, D and E. This will make it easier to duplicate values.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/28_spreadsheet_2.png)

Go back to the top of column B with `ctrl+up arrow`. Enter the following formula into the first cell of column B: `=concat(A1," - ")`. Use `ctrl+shift+down arrow` to select the whole column B, then use `ctrl+d` to duplicate the formula into all the cells of column B.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/29_spreadsheet_3.png)

If you still have your JSONPath.com window open, go back to that. If not, follow Step 1 again. You should have valid (for our purposes) JSON in the 'Input' window. This time, enter `*.*.jyutping` into the JSONPath input bar above.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/30_JSONPath_2.png)

Copy the resulting Jyutping with `ctrl+a`.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/31_copy_jyutping.png)

Paste the Jyutping into column C. Then type in the following formula into column D: `=LEFT(C1,LEN(C1)-1)`, in order to get rid of the comma at the end of our Jyutping. Use `ctrl+shift+down arrow` and `ctrl+d` to duplicate the formula into all of the column's cells. 

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/32_spreadsheet_4.png)

You'll notice however, that the very last cell does not have a comma, so we've overridden the tone. Copy the value from the last cell of column C, into the last cell of D.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/33_spreadsheet_5.png)

Use the following formula in the first cell of column E: `=concat(B1,D1)`. Use `ctrl+shift+down arrow` and `ctrl+d` to duplicate the formula into all of the column's cells.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/34_spreadsheet_6.png)

While you've still got the whole column selected, copy the values and `ctrl+shift+p` paste them into a word program of your choice. Feel free to increase the font size.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/35_word.png)

You'll probably want to break this word file up into chunks that correspond to your .mp3 files. I have 5 of these files for each of mine. It'll take some trial and error to find where each chunk should begin and end. If you'd rather skip this, you can make use of the audio files and word docs that I have prepared.

If you're like me, you may also want to keep track of which drills you've completed using a spreadsheet, see mine below.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/36_drill_tally.png)

I like to do my drills, listening and repeating, on my phone. I have a Google drive widget that goes straight to the folder with the drills.

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/37_gdrive_widget.png)

![](https://github.com/CallumDyer/Scraping-the-Cantolounge-Jyutping-Chart/blob/main/Screenshots/38_cantonese_tone_drills_folder.png)
