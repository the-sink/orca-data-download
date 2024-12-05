# orca-data-download
Instructions on downloading usage data for managed ORCA cards

After trying to use [moshobo/orca-wrapped](https://github.com/moshobo/orca-wrapped), I noticed on the myORCA website that managed/load-only cards provided by employers, colleges, etc do not let you view usage data on the website. However, I was curious if this data was still technically accessible to the user but just not visible on the site. As it turns out, it is! (at least, it was in my specific case and I hope this works for others too...)

This guide assumes you are using a Windows device and the Google Chrome web browser as that is what I'm using here, however you may be able to adapt this to other machines and browsers on your own. To do this, you will need to use your browser's developer tools/"inspect element" mode to view the network requests being made by the site.

The easiest way I've been able to do this is to have at least TWO cards on my account already: the managed card I'd like to download the data of, and a personal card that I can already download the data of on the myORCA website without issue. Ensure this is the case on your account before starting.

Here are the steps I took (and you can take) to download your data:

## 1. Identify Transit Account ID

Navigate to the My Cards page on the myORCA website. It should show you a list of cards you have added to your account.

![](https://i.imgur.com/wwXhouY.png)

Open your browser's developer tools. In Chrome, you can press ``Ctrl+Shift+I``; a window will appear on the right-hand side of the page. Navigate to the "Network" tab. Reload the page to see network requests made by the page. Somewhere in the list you should be able to find a request titled ``GetTransitAccounts``.

![](https://i.imgur.com/eUujUXC.png)

Click on that request. To the right, ensure you are on the ``Response`` tab.

![](https://i.imgur.com/4TeIhWY.png)

The data displayed here in JSON format is a list of "transit accounts" associated with your myORCA account. The data within the "Result" field is relevant here. It will take the form of something like this:

```json
"Result": [
            {
                "Id": #######,
                (...)
                "PrintedNumber": #######,
                (...)
                "AssociationDescription": #######,
                (...)
            },
            {
                "Id": #######,
                (...)
                "PrintedNumber": #######,
                (...)
                "AssociationDescription": #######,
                (...)
            },
        ]
```

With one entry for each card on your account. The most helpful fields are shown above: ``Id``, ``PrintedNumber``, and ``AssociationDescription``. Use either the ``PrintedNumber`` (actual card number) or ``AssociationDescription`` (name of the card on your account) to identify which entry is the one you're looking for. Once you have found it, copy down the ``Id`` number associated with it and keep it somewhere safe. Importantly, this ID number is **not** the same as the card number printed on the physical card itself.

## 2. Make a request to a non-managed card

Now you will need to get it to make a request for a non-managed card that retrieves its activity data. This step only exists so you can copy the request, modify it with the new ID, and send this request again separately to get data on the *other* card.

Keep the network tab of the developer tools open, and locate the personal card. Click "Manage card" and then switch to the "Card Activity" tab.

![](https://i.imgur.com/ngnMLPU.png)

In the developer tools, you should notice somewhere near the bottom of the list that a few ``GetTransactionHistoryTable`` requests have been made. It appears that usually the second request on the list is the one you're looking for, but to make sure switch to the "Response" tab like you did earlier and check both; the one with more content is the one you need.

Once you have identified the relevant request, do the following:

Right click > Copy > Copy as cURL (cmd)

![](https://i.imgur.com/53KuCNI.png)

Now, there will be a command in your clipboard that can be run to make this HTTP request again.

Don't close this page on your browser; leaving the page open will keep your session active. If your session expires, you will need to restart this process. You may also need to occasionally tab back into the site to click on a popup that keeps your session alive.

## 3. Save and modify this command

Open notepad (or a text editor of your preference) and paste the command that was copied into your clipboard from the browser.

![](https://i.imgur.com/1UTuum9.png)

The very last line of this command is the relevant one. It will look something like this:

```cmd
  --data-raw ^"TransitAccountId=0000000^&SortField=^&SortDescending=false^&From=^&To=^&Skip=0^&Take=10^"
```

In this line, change ``TransitAccountId=000000`` (whatever number it was) to the ID you saved earlier. Additionally, remove the number from ``Take=10^`` so that it is just ``Take=^``.

Finally, at the very end of this line, add a space followed by a caret (``^``). And on a new line below it, add the following:

```cmd
  -o ORCAHistory.json
```

It will look something like this at the bottom of the file:

```cmd
  --data-raw ^"TransitAccountId=98765432^&SortField=^&SortDescending=false^&From=^&To=^&Skip=0^&Take=^" ^
  -o ORCAHistory.json
```

The command should now be ready. Save it somewhere as a ``.cmd`` file (for example ``collectData.cmd``).

## 4. Run the script, convert the data to CSV

Double-click on the script and, assuming everything was set up properly, it should download the history data to the same folder the script is located in, as "ORCAHistory.json".

This repo contains a converter you can use in your browser to generate a CSV file from this data. Open the following: [https://the-sink.github.io/orca-data-download/convert.html](https://the-sink.github.io/orca-data-download/convert.html)

Using the "Choose file" button at the top left of the page, select your ORCAHistory.json file. It should almost immediately afterwards download a file named ``Card-Activity.csv``. This will be in a similar format to the CSV files you would download from the myORCA website directly (with a few omissions because I'm both lazy and confused :>).

You should be able to use this CSV file in the ORCA Wrapped tool now!