This is the README for Redirect Manager.

**************************************************************************************************************************
Source code coming soon. Must wait on my Lead Developer and Senior Developer to get back to me with the code that does not have company information with it.
**************************************************************************************************************************


Before starting, I highly suggest downloading this and reading it within a text editor, or whatever other software you use to read documents. 

Redirect Manager is a project that I had worked on during my internship at Citrix Systems, Inc during the summer of 2017 (May 2017 - July 2017) with direct help from my Lead Developer, Phani Jasthi (https://www.linkedin.com/in/phanijasthi/), and my Senior Developer, Raghu Varma (). 

This was written in Node.JS, and this is the source code for it. In order to fully implement the project, Jira, Apache and Chef software are also utilized. 

**************************************************************************************************************************
Overview of the software:
    
Redirect Manager is a tool that automates the process of creating a redirect. 

As mentioned earlier, Jira, Apache and Chef software was used in the process of making the Redirect Manager go live for use by other members of the company for the Digital Marketing team. These will be covered in detail below.
**************************************************************************************************************************

**************************************************************************************************************************
Jira: 
    
The first step is using this code is making sure that you have a Jira board that it can connect to. On top of being the tool where a user creates the tickets to submit a redirect request, the Jira board is also the source of truth for the redirects that are currently live on the web, while giving other functionality on top of that as well. I will go in depth about the setup we used during my internship in order to help with such process in case that it is needed.

Jira is a software created by the developers at Atlassian. A nice software to use for multiple purposes as the boards that they allow you to create can extend beyond this Redirect Manager if you had not known already. As a result, there are tutorials that Atlassian has in order to help with the creation of a Jira board (nice starting point: https://confluence.atlassian.com/agile/jira-agile-user-s-guide/creating-a-board).

With that said, for our set up, we had 4 statuses that were part of the workflow. The 4 statuses that we used were called Open, Push, Live, and Sunset. On the board, we only show the statuses Open, Push, and Live while keeping Sunset hidden. Now, when a new ticket is created, a form should pop up asking for relevant information related to the redirect request that is about to be created. In our setup, the fields we use are named Summary, Redirect From for the source URL that the redirect will happen from, Redirect To for the final destination URL that the Redirect From will go to, Start Date for the date a person wants the redirect to go live, and End Date for when a user wants the redirect to be pulled from the web. There are other sections that are more self-explanatory. Check the screenshots folder for an example of our form (the file name is "FormExample.png"). As you will see, there are hints that state for the Redirect From that all URLs must be relative. For our implementation, all redirects were to happen from internal sites for the company. The code base can be only changed in order to implement your own companies assets and needs for that section. With that being said, I shall remind that this project pertains to solutions needed for the company, and is not a 100% general solution to uploading redirects. This source code simply gives a good starting point in order to be implemented for use if need be by others. Moving on from that, once a ticket is done and all the fields are filled out, the user then hits the create button. For our implementation, we send the created ticket to the Push status. It as at this step that a webhook is triggered that connects to the NodeJs app, the source code is given in this repo and is run. It is at this point the backend validations occur. This is handled mainly within the JiraController file. If validations need to be changed for your company, I would suggest doing those changes there. If a ticket fails validations, then it will go to the Open status where the program writes a comment on the ticket stating why it failed. Once the changes are made and the ticket has been properly edited, then a person must transition that corrected ticket back to the open status in order so validation checks can be run again. If validations pass, then the program will write that the ticket has passed validation checks. It is at this point that the ticket waits for the cron job toLiveJob() to run. This is what sends proper tickets to the live status. On top of that, there is another cron job that runs called sunsetJob(). This one checks for end dates to see if a ticket has expired and move that ticket to the Sunset status so that the ticket and its corresponding redirect no longer exist. This gives the overview for the statuses and how they work.

For the transitions, we had one for Open to Push, Push to Live, Live to Sunset, Live to Open. You can add more transitions if you want for whichever status-to-status you want. The transitions are key. The reason being this is where webhooks are set. In the above paragraph, I spoke on the webhooks a bit. Basically, a webhook listens for certain events, and once the webhooks are triggered due to that event, it triggers another event at a location listening to the webhook for certain code to run (here's a better explanation: (https://webhooks.pbworks.com/w/page/13385124/FrontPage). During testing, Ngrok was our best friend. Here's a link for what ngrok is about: (https://ngrok.com/). Basically, ngrok gives you a link that listens for webhooks and sends the post to the program so that the code can be run. To use ngrok, you simply put the executable in a place you can always find and on a terminal, run "./ngrok http 'port number'", where 'port number' is where you want to ngrok to listen to. So if you want ngrok to listen to your 3000 port, you'd write "./ngrok http 3000". It is through this that validation steps were able to occur because we set a webhook on the transition from Open to Push status so validation could take place. Now if you read the previous paragraph, you may remember the mention that when a ticket is created, it starts off by directly starting in the Push status. When we implemented this, we realized that the webhook for Open to Push was no longer triggered. As a result, we needed to find a work around so that the validation gets triggered for the newly created ticket. In truth, Raghu Varma, mentioned in the beginning, handled this workaround, so I do not know of his exact solution towards it. Regardless, your Jira board does not have to follow ours. It's simply a skeleton for your implementation. So if finding the workaround is not worth the effort, you can just have the user transition from Open to Push, or whatever you wish to call your statuses, and it will work just as well. 

With that, the Jira part is pretty much wrapped up. 
**************************************************************************************************************************

**************************************************************************************************************************
Node.JS Source Code:

This portion is everything that pertians to the code found in the repo. 

To start, I must say that the intial starting point to the application is running Express at the very beginning. Here's a link to ExpressJS (https://expressjs.com/). Express is useful as it gives a nice starting point to start coding for projects that are beyond the scope of this project as well. To walk you through many of the files, I will start from the top-down where files are ordered alphabetically. 

At the beginning, the bin directory. With the files that express generates, this is the one that holds only one file named www. If you're using the WebStorm IDE and want to run this app through NodeJS, then you have to go into the WebStorm IDE configuration and set the "JavaScript File" field to bin/www. This is pretty much all this is good for. If you're not using the WebStorm IDE, I am sure that it is still similar if the IDE allows you to run the program within it. 

Secondly, comes the config file. In it, you will see the files index.js, config-dev.js and config-prod.js. Within config-dev.js and config-prod.js are the configurations for how the application should run and holds certain important information. The difference between the dev and prod config files is that when a person runs this program, they must specify if they are running a development run, or a run that should be in production. This is determined by the Process environment functionality that comes with Node.JS. When running the app from the terminal, you write "export NODE_ENV='environment to run'". For example, if you want to run the development environment, you would write "export NODE_ENV=DEV". Now within other files that we are yet to get to, the config files are needed. Within those files, "require" is used in the following way: "config = require(../config)". This is where the index.js file comes in handy. For Node.JS/ Express, if a file in a directory is required but a specific one is not specified, the program will look for a file named "index.js" automatically. Within our index.js file, we check the node environment by using process.env in order to get the proper configuration file each time. The use of process.env is the reason we have to do "export NODE_ENV='environment to run'" when running the program. Again, I highly suggest running this through the terminal. I was not able to figure out how to set node environments on the WebStorm IDE directly but instantly worked once I tried it on the terminal and ran node through terminal. Now, within the config files, you'll see 2 JSONs named who are embodied by, "jira" and "git". In the jira JSON, you put everything related to your Jira Board. username to login, host, port number, the custom fields on the tickets for the data you want to extract, and the transitions on your Jira Board. Again, this is not something you have to follow, this is simply how we did it during our implementation. In the git JSON, I put all the information related to the git repo that holds the location of the text file that holds all current redirects. While the Jira board is a source of truth for the live tickets, we also decided to have a text file in a repo with the current live redirects so that they could all be looked at more easily. Within the git JSON, we held the user email, username and git repo link in order for use when it came to cloning and pushing that redirects file. There are also 3 other variables named logDirectory, logFileName and execptionLogFile. If you are using logs for your program, which I highly suggest, this is where you'd put that information.

Next we see the controllers. In this directory, we see gitController and jiraController. These are the driving functions that connect to gitServices and jiraServices within the services directory, respectuflly. Within the gitController is the process for updating the redirects file and calling the git terminal code for cloning and pushing of the updated file. when you open jiraController, you will probably notice that this is where the bulk of functionality for the program occurs. In it holds the validator function, and all of it's helper functions. This is also where you can find the cron jobs for transitioning tickets from Push to Live and Live to Sunset. Here is the following documentation for cron: (https://www.npmjs.com/package/cron). As mentioned eariler, if there are any changes that need to be made for validation or git flow, I suggest working mostly in this directory for it. 

After the controllers directory, we have the Logs directory. Within it is where your logs will go pertaining to exceptions and non-exception logs. This will connect to the file name you put within the config file. Using the information within those config files is what determines the name of the file that will hold the information for logs occuring as the program runs. Now as a warning, this is within the gitignore file, but I do find it very important as it helps with debugging for possible problems for when the program runs. Especially for when it breaks, it allows you to see approximately where it broke. 

After logs, you'll see the node_modules directory. This simply holds all of the modules that are used by the program. I suggest not playing with this directly. If you need to add a module, I highly suggest doing npm install. 

After node_modules, you'll see the public/stylesheets. This directory does not mean much to the program. Created due to Express. 

Now comes the routes directory. Within it is one lonely file named redirect.js. This is how the program knows what webhooks to listen to based off the relative path given. You may wonder what it is relative to. For testing purposes, before putting it on the server at least, it was relative to the URL that is produced by running the ngrok executable. If you're not using the ngrok, it will need some relative path from some URL in order to run. Main reason is that this is how the webhooks interact, which is key for this software to work as expected.

Next up is the services directory. Within it are gitService.js, jiraService.js and loggerService.js. jiraService.js holds all functions relating to jira API functions. This is what transitions tickets and adds comments to tickets on the Jira board, You can add your own jira functions with the Jira API that you need as you see fit if the ones there are not enough for what you need done. In the gitService file holds all the functionality that relates to writing commands on the terminal for git functionality like cloning, deleting and pushing the updated redirects file. The loggerService file simply holds the set up for how the logs will work within the program. 

With the last directory, views, it's something you needn't really worry about. This directory holds the pug files that generates HTML and CSS for the app. Since this app doesn't realize utilize it's own space and domain, this isn't really important and there since Express created it upon its generation.

As expected, the .gitignore file comes next. There's a myriad of explanations for this one already.

Now for app.js is how the program knows what routes to use and what to do on exceptions and such. This doesn't really needed to be touched much except if you need to add a new route. 

Then comes the package.json file. Within it holds the dependencies used within the project. This file is very important for the modules within it are needed in order for the program to run, to begin with. Modules saved within this file can easily be installed with the command "npm install" on the terminal.

Finally, the README.md, and you already know what this is.
**************************************************************************************************************************

**************************************************************************************************************************
Chef:
    
Chef is a software program that helps automate IT infrastructure. In truth, I did not touch this during my internship, and have very little knowledge. The extent on what I know of Chef is that this is the program that would check the currently live redirects and actually implement the 301/302 HTTP status code for it. This is something that my Senior Developer and Lead Developer handled.
**************************************************************************************************************************

**************************************************************************************************************************
Server: 
    
After testing with ngrok, it was necessary that we put the code on a server so that the code does not have to constantly run on the local computer. So unless you have a computer that you can dedicate to running the program 24/7, I'd look for a server to put this on. Again, I have little knowledge on this component on this piece of the project for my Senior Developer and Lead Developer mainly dealt with this. 
**************************************************************************************************************************

**************************************************************************************************************************    
Apache:

Apache is a Web server software that is used within the overall product of Redirect Manager. I had no interaction Apache throughout this proecss, and this was left up to my Lead and Senior developers. I simply know that it plays a role with the overall program.
**************************************************************************************************************************

With that, everything is wrapped up. As I mentioned before, I would really like to stress that this is not a 100% general software program for having redirects go live. This is a software program that I had created, along with help from my Senior Developer and Lead Developer, in order to find a solution for making redirects go live quicker within the Citrix Systems, Inc. internal sites. This should serve as a skeleton and be expanded upon. At the moment, I have no plans to continue development on this project. I am putting this up for proof of work that I have done, and so that it can help anyone who may possibly need such a program themselves. 

As mentioned above, but to make the point clearer, I would love it for people to take this and improve on it as they see fit, and to use however they see fit. 

Happy coding!
