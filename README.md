# OpenHAB controller script for PixelIt
This script handles communication between [OpenHAB](https://www.openhab.org/) and the PixelIt device (see [o0shojo0o/PixelIt](https://github.com/o0shojo0o/PixelIt) and https://www.bastelbunker.de/pixel-it/).

It allows both sending messages to PixelIt for immediate display and also managing a playlist with screen definitions, thus making OpenHAB become a PixelIt controller.

## How it works
### How does PixelIt work?
In order to display content on your PixelIt device, you have to send a `screen definition`. A `screen definition` is a piece of JSON code describing what to display next:
```JSON
{
   "switchAnimation":{
      "aktiv":true,
      "animation":"fade"
   },
   "bitmap":{
      "data":
[0,32799,63519,43029,43029,43029,0,0,0,32799,63519,0,0,0,63504,0,0,32799,63519,0,0,0,63504,0,0,32799,63519,0,0,0,63504,0,0,32799,63519,43029,43029,43029,0,0,0,32799,63519,0,0,0,0,0,0,32799,63519,0,0,0,0,0,0,32799,63519,0,0,0,0,0],
      "position":{
         "x":0,
         "y":0
      },
      "size":{
         "width":8,
         "height":8
      }
   },
   "text":{
      "textString":"10",
      "bigFont":false,
      "centerText":true,
      "hexColor":"#FFFFFF"
   }
}
```
This tells PixelIt to...
* ... fade the current screen out and the new screen in
* ... show a bitmap described by the `data` section (actually, that's the PixelIt logo)
* ... show the text `"10"` next to it, centered, in white color

There are more commands to use. See the full [API specification](https://docs.bastelbunker.de/pixelit/api.html).

### How does this controller script work?
Basically, the "controller" helps you to manage screen definitions like described before. It stores them in a single playlist and plays them out to your PixelIt one after another.

The "controller" consists of
* an OpenHAB String item receiving commands
* a script being triggered by comamnds sent to this item, manipulating playlist items.

## Installation
### 1) Item
Create a String Item which will receive your commands. Choose any name that fits your naming convention for this item:

```Xtend
String PixelItCommand "Commands for PixelIt controller"
```
(Of course you can also use the Main GUI to create the String Item.)

### 2) Rule
Create a rule to handle commands that are posted to the item.
1. Create a new rule. Pick any name and description you like.
2. From the first screen, select the `Code` tab and paste the content of the YAML-file from this repository.
3. Make sure to insert the name you chose for the item and the proper IP address of your PixelIt at the places indicated:
![image](https://user-images.githubusercontent.com/71180164/145290385-04657deb-6702-4069-b23a-21d0375ccb9f.png)

That's it! Now start sending commands to your PixelIt controller:
```Xtend
PixelItCommand.sendCommand("message Hello World!")
```

## Usage
### Creating screens
The following commands are supported:
```
add SCREENNAME SCREENDEF
update SCREENNAME SCREENDEF
```
where `SCREENNAME` is any name you want to give that screen for managing it in your playlist, and `SCREENDEF` is a screen definition like described [above](https://github.com/miccgn/openhab-pixelit#how-does-pixelit-work). `add` and `update` are completely interchangeable: if the screen named `SCREENNAME` already exists, it will be updated, otherwise it will be created.

Example:
```
add welcome {"switchAnimation":{"aktiv":true,"animation":"fade"},"bitmap":{"data":[0,32799,63519,43029,43029,43029,0,0,0,32799,63519,0,0,0,63504,0,0,32799,63519,0,0,0,63504,0,0,32799,63519,0,0,0,63504,0,0,32799,63519,43029,43029,43029,0,0,0,32799,63519,0,0,0,0,0,0,32799,63519,0,0,0,0,0,0,32799,63519,0,0,0,0,0],"position":{"x":0,"y":0},"size":{"width":8,"height":8}},"text":{"textString":"Welcome!","bigFont":false,"centerText":true,"hexColor":"#FFFFFF"}}
```

As soon as the first screen has been added, a timer will play the playlist.

### Defining for how long to display each screen
By default, each screen will be displayed for 7 seconds before going on with the next item in the playlist. You can define this team per screen with an additional item in your screen definiton:
```
... "displayTime": SECONDS, ...
```

Example:
```
add welcome {"displayTime": 10, "switchAnimation":{"aktiv":true,"animation":"fade"},"bitmap":{"data":[0,32799,63519,43029,43029,43029,0,0,0,32799,63519,0,0,0,63504,0,0,32799,63519,0,0,0,63504,0,0,32799,63519,0,0,0,63504,0,0,32799,63519,43029,43029,43029,0,0,0,32799,63519,0,0,0,0,0,0,32799,63519,0,0,0,0,0,0,32799,63519,0,0,0,0,0],"position":{"x":0,"y":0},"size":{"width":8,"height":8}},"text":{"textString":"Welcome!","bigFont":false,"centerText":true,"hexColor":"#FFFFFF"}}
```
This will display the screen named `welcome` for 10 seconds before advancing to the next one.

### Removing screens
```
delete SCREENNAME
```
Removes the screen defined with the name `SCREENNAME` from your playlist.

### Interrupting with urgent messages
```
message SCREENDEF
```
Shows the screen defined by `SCREENDEF`, then goes on with the next regular screen in your playlist. This is intended to be used for one-time messages from your rules.

```
error TEXT
warning TEXT
```
These two commands go without a full screen definition. Instead, they just take any text as parameter, wrap it with a default screen definition and use the technique for the `message` command to display it.
