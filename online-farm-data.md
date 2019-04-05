# Having fun with Farming Simulator and Google Cloud

// TODO Link to steam

Enjoying the work I do is very important to me.   
There always has to be a challenge in it, but having fun with colleagues is also a big factor.  
I love to dig deep into a subject sometimes to master it and teach other what I've learned.   
Not always though, because sometimes I really enjoy building useless software.   

At work we were recently discussing games and the possibility to play together.   
When they asked me about the games in my Steam library, I said "Well, amongst other things, I have **Farming Simulator**!"   
You should have seen the look on their faces.   
First they thought I was crazy, but to me it is a game that you can play without the pressure to "win".   
After a day of work I'm not in the mood for a shooter that gives you an adrenaline rush.   
I rather enjoy a *slow game* that really takes away stres.  
Farming Simulator is great for this and their latest edition **Farming Simulator 19** is a great piece of work!  

![Farming Simulator 2019](online-farm-data/fs19_headliner.jpg)

In the end, my colleagues got excited and also bought the game.   
We decided to rent a dedicated Farming Simulator server, so we could play together.   
The server is always online so even if the others aren't online, I can still farm the digital lands.   
It is password protected, so nobody can mess up the progress.   
Our farm currently has tractors of multiple manifacturers and we specialized in horses and forestry!    
We have a lot of fun riding the tractors and build up our farm together.   
The physics aren't super realistic, which brings a lot of fun sometimes.   
Driving a combine harvester off a cliff and then driving on like nothing is wrong, why not?

![Physics in FS19 can be really fun](online-farm-data/fs19_physics.png)
**not my screenshot*

##### Then we found something really cool
Our rented farm server comes with an administrator dashboard.   
It allows to configure some server settings, add mods to the game or restart the server.   
There is also an app for Android and iOS, which mostly has read-only functionality.   
At some point I was clicking through all the configurable options and really found a gem!

The server offers an endpoint that offers real time game statistics in XML or JSON format.   
Futhermore, there were PHP based plugins and libraries to read real time data and display it on websites.   
The API is most likely used by the mobile apps, but it was publicly accessible from the link in the admin panel.   
Clicking the link showed me XML output, but changing the URL a bit, I found that it would also return JSON!   
This is great, because it makes processing it easier.   
The only downside for me was that it does not collect statistics over time.   
It will return the current state of the server & who is online right now, but nothing to make cool dashboards out of.

### Harvesting data
// TODO
