A "design document" for your project in the form of a Markdown file called DESIGN.md that discusses, technically, how you implemented your project and why you made the design decisions you did. Your design document should be at least several paragraphs in length. Whereas your documentation is meant to be a user’s manual, consider your design document your opportunity to give the staff a technical tour of your project underneath its hood.

Welcome to the hood, or rather the under the hood (this isn’t the ghetto). Join me as I take you along a journey showing you the inner workings of a product that caused midnight tears, 3 AM cries of joys, and many yawns of utter exhaustion.

I’m first going to show you around this thing called application.py. I just wanted to preface that we had based our application.py on the hunk of code found in Finance’s application.py. However, we implemented many of our own functions that makes it quite nothing like Finance. I figured that a good way to go through this would be to examine each function in each app.route.

First up to base, we have @app.route(“/“) that has def index(), which simply renders index.html, our homepage that also serves as an ‘About’ page. This is the only page that is not @login_required (we’re all about those inclusive social spaces)!

Next is @app.route("/home") which has def home(). This function redirects to the main user page, which are different for the donors, which in this case is most likely solely HUDS (explained on our Documentation), and the shelters. This is done by examining the logged-in user’s ID, which is stored in the “users” table in the SQL database. If the ID of the logged in user corresponds to a donor, the webpage directs them to donor home page (huds.html), and if the ID corresponds to a shelter, the webpage directs them to the shelter’s home page (shelter.html).

Then, we have @app.route(“/huds”) which has def huds(). This just renders huds.html, which is the HUDS/donor user home page. Now, in this part of application.py, we layout the functions of the HUDS/donor user’s interface.

In @app.route(“/donationform”) which has def donationform(), the donation form, donation.html, is requested via GET. When the form is submitted by HUDS/donor via POST, then a couple steps occur. First, all the numbers from the form is inserted into the “huds” table in the SQL database. Afterwards, the user is redirected to the HUDS homepage. However, if the form was not completely filled out, then an apology is rendered, asking the user to fill out the form in its entirety.

Now, we have @app.route(“/donatenow”) which has def donatenow(). Just a heads up, this is the longest section, but also the most important so try to stick it through with me. This app.route and its associate function “donatenow” is accessed via GET when HUDS clicks on the “Donate Now” button on the user’s home page. When this is clicked several things occur:

	1. The “huds” table in the SQL Database is checked to see if it is empty. If it is, then HUDS had not yet added any donations so the web program raises an apology, asking HUDS to make a donation.
	2. After HUDS had made some donations, the sum is found of all the number of servings in each food category for that day, which was in the “huds” table. These summed up values are then put into a list called “huds_list.” We then duplicate “huds_list” by setting “huds_initial” equal to “huds_list.” This allows us to display the amount of food HUDS donated for the day, while giving us another list to use for other uses.

Then, when HUDS clicks the “Deliver” button at the end of the donatenow.html, a few more steps occur that are outlined in this function:

	1. Today’s date is noted.
	2. Looking at the various shelters’ advance requests for that day, a sum is taken of all the food values for each food category.
	3. If no requests were made, then we raise an apology informing HUDS of this.

Then, in the next part of this function, we initialize many lists and dictionaries to be used to store the results of our calculations of the utility, which we describe as “happiness” value. Iterating over the number of shelters that have requested for that day, we assign a “happiness” value to each shelter. This value is calculated by dividing (the greatest amount of food that both a shelter requested and HUDS could donate) by (the distance of the shelter from HUDS), and then taking the square root of this quotient. This “happiness” value prevents someone from requesting very far away or too much food relative to HUDS. (A quick side note: the MapQuest API used is not perfect in that, the API perfectly calculates the distances between 2 real addresses, but it also accepts fake addresses and assigns a distance to the fake address.)

Moving along, after assigning “happiness” values to each shelter that requested for the specific date, the shelters are ranked by their “happiness” value and then up to 3 shelters with the highest “happiness” values are “matched” with HUDS, who would donate that day’s food to those shelters.

In order to decide how much to donate to each shelter, this function tries to match the shelter’s requests to the best of its ability. So the first shelter is given as much as possible in each food category, up to the amount that they requested or the amount that HUDS has to offer. The food amount that is leftover is then allocated to the second shelter, up to the amount that they requested or the amount that HUDS has to offer. And the final shelter then receives the amount of food they requested or the amount that HUDS has to offer. If there are any remaining foods afterwards, they are discarded.

We decided to do this “trickle down” distribution of food because we wanted to distribute to more than one shelter. We decided that choosing three shelters would allow for the shelters to have at least most of their needs matched, even if it meant that some shelters won’t be able to get the meals. In the future, we can expand this program by creating a more complex algorithm could handle many, many shelters

The final part of this function calculates the remaining HUDS inventory after making the donations for the day. This calculation then updates amount left in HUDS’ inventory, the dictionary containing all the donations, and the list of name and addresses of the shelters that were donated to.

Whew.

The final part of the HUDS’ interface is @app.route(“/deliver”) which has def deliver(). This function simply clears everything in the “huds” table in the SQL database when HUDS presses “Deliver” in the donatenow.html, and then leads to deliver.html, which thanks HUDS for donating.

We recognize that by clearing everything in the HUDS database, we clear the inputted food donations of all the donors in our program; our program currently is geared only towards HUDS (i.e. HUDS is the only donor, while many requesters (shelters) can exist). Also, because we have not gotten the chance to update the requests table after a delivery is made, our web application is restricted to one donor making one donation a day.

So that was the application.py describing homepage (index) and the HUDS’s interface. Now, we’re going to look at the Shelter’s interface and then other sections of application.py.

---

The first part of the Shelter’s interface is @app.route(“/shelter”) which has def shelter(). This simply directs users to shelter.html.

In @app.route(“/requestform”) which has def requestform(), all the amounts of food that the shelter requested in inputted into the “requests” table into SQL database. It also ensures that the date of the request is not in the past or today, checking to make sure that all requests are in the future. This is because while HUDS is deciding who to donate to, we don’t want to have to close the request form so that no more requests are made during that time (because shelters do not know when HUDS will decide to click Donate Now). To avoid having to close and open the form (which would require live updates when HUDS clicks Donate Now), we simply require shelters to request in advance. After a shelter makes a request, the ‘requests’ table in our database will be updated to show how much the shelter requested. Then, a line of code indicates that requestform.html is display if called via GET.

Moving along, the @app.route(“/viewrequest”) which has def viewrequest() finds the requests associated with the logged in shelter by going back into the “requests” table and looking for the logged in user’s ID and then printing out all the requests that they ever made. Like ever made.

Great! So we just went through the Shelter’s interface of application.py. Next up, we have @app.route(“/login”) which has def login(), which is one of the few survivors from Finance and has not changed.

We’re almost there… (trust me, writing this is nearly as bad as having to read it (probably worse)). So the @app.route(“/register”) which has def register() has remnant parts of Finance, in that, the first part relating to username and password is from Finance. In the next part of this function, we get the address of the shelter and whether or not the registering user is a donor or a requester from the registration form and then insert those pieces of information into the “users” table in the SQL database. If the form isn’t filled out completely, we apologize and ask them to PLEASE FILL OUT THE FORM COMPLETELY BECAUSE WHY WOULDN’T YOU??? Then, via GET, we render register.html.

Nearly there, @app.route(“/logout”) which has def logout() and def errorhandler(e) is a near replica of Finance’s logout distribution code with just a redirection to the @app.route(“/home”).

Finally (and I mean, FINALLY), last but definitely not least, we have @app.route(“/check”) which has def check(), which is from Finance, and def get_distance(address1, address2). def check() gets the username from the register form and checks if that username exists in the users database already; it returns, in JSON format, a boolean corresponding to “true” if the username is available, and “false otherwise”. Lastly (f’realsies this time), def get_distance(address1, address2) uses the MapQuest API to calculate the distance between 2 addresses.

***END OF Part 1: application.py***
Hello.

It’s me.

I was wondering after all these words you’d like to read more ):)


***Part 2: Everything Else***

So… I know you must not be too happy with me… but we’re not done yet. We have some more to talk to you about. But to make your life easier, I’m going to give quick and nice descriptions and mini explanations to the web application’s HTML pages and our hudsmatch.db.

Sparknotes: THE HTML PAGES
apology.html was taken from the finance problem set, and simply displays the error message if the user tries to execute an action our web application does not support.
deliver.html is simply a thank you page displayed to HUDS after they donate their inventory based on our recommendations; it also contains a button that takes HUDS back to the donation form so they can continue inputting inventory if necessary.
donatenow.html contains three different tables: the topmost table displays HUDS’s original inventory, the middle table displays the number of items the web application recommends HUDS to donate to the top shelters, and the bottommost table displays HUDS’s leftover inventory after making those donations.
donationform.html contains a form where HUDS can input leftover inventory.
huds.html is the homepage of donors, welcoming them; it contains two buttons: a button that takes to a form that allows HUDS to add leftover inventory to the database (takes them to donationform.html), and another button that allows HUDS to view the top shelters they should donate their leftover inventory to (takes them to donatenow.html)
index.html is our general homepage when someone visits our website, and is also the webpage linked to “About” in the navbar.
layout.html is our layout template that we extend to each webpage.
login.html is our login page.
register.html contains our register form for new users.
requestform.html contains the form where shelters can make donation requests.
requestthanks.html contains a page confirming that a shelter has successfully submitted their donation request, and contains two buttons -- one that takes the user back to requestform.html so they can make another donation request, while the other takes the user to viewrequests.html where they can see a table listing the history of their requests.
shelter.html is the homepage of shelters, welcoming them; it contains two buttons: a button that takes to a form that allows shelters to request food donations (takes them to requestform.html), and another button that allows shelters to view their past requests (takes them to viewrequests.html).
viewrequests.html contains a table listing the history of their requests.

hudsmatch.db
The huds table contains seven columns, and stores the number of food items HUDS had in leftover inventory that they had inputted into the database via the form in donationform.html.

The first column is the id of the donor, while the other six columns contain the amounts donated of each food category (soup, salad, sandwich, entree, side, and dessert).
The requests table contains eight columns, and stores the number of items in each food category each shelter had requested for delivery via the form in donationform.html. The first column is the id of the requester, the date they would like a certain requested food bundle to be delivered, and the remaining six columns contain the amounts requested of each food category (soup, salad, sandwich, entree, side, and dessert).

The users table contains five columns, and stores the information of each user of our web application. The id column stores their unique IDs (and is thus a primary key), the username column stores their usernames (and is a unique value so no two usernames can be the same), hash stores their passwords after hashing, address stores their addresses, and party records whether a user is a donor or a requester. We decided not to create separate “users” tables for donors and requesters because we realized that we can identify the different parties with boolean variable called “party”, setting donors=1 and requesters=0.

***END OF Part 2: Everything Else***

So thanks. Thanks for going through this “nice and simple” tour of our design. Thanks for holding in there and reading everything. I’m impressed. Like actually impressed. And if you didn’t read everything, then I hope these compliments make you feel slightly guilty (not really though, I understand). Good luck with your own finals, and I hope you have a great break!
