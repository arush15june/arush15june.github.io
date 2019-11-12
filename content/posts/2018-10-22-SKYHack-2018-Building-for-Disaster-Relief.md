---
title: "SKYHack 2018 Building for Disaster Relief"
published: true
date: 2018-10-22T00:00:00.000Z 
categories: ["hackathon"]
tags: ["skyhack", "chhattisgarh", "incubator", "hackathon", "android", "react", "react-native",]
---

# SKYHack 2018: Building for Disaster Relief

This blog post details my experience in participating in [SkyHack 2018](skyhack.36inc.in), a hackathon by the Chhattisgarh Government to build applications for Android Phones going to be distributed as part of the [Sanchar Kranti Yojana](https://navbharattimes.indiatimes.com/business/business-news/micromax-jio-bag-rs-1500-cr-order-from-chhattisgarh-government/articleshow/65830455.cms).

The process of participation included selecting a social problem to solve from a set of 6 statements, submitted a proposal of our solution before 31st July 2018, Implementing a PoC of the solution and pitch our solutions at 36inc, Raipur.

I participated in a team of 4, with, [Vinayakk Garg](https://github.com/vinayakkgarg), [Hardik Khurana](https://github.com/hardik0330) and [Rishab Bansal](https://github.com/rishab-rb).

# A Problem to Conquer

Out of 6 problem statements, dealing with different problems, We were intrigued by with an idea for the 3rd one

_During an incidence of natural disaster, information and time are two of the most crucial resources. So, can we source information from citizens and simultaneously disseminate it among them in an effective and targeted manner, to minimise damage and save lives?_

The problem was divided into two parts, in the first part we had to build a disaster education platform for educating people before a disaster strikes and a system to provide relief to people under duress.

Another major requirement of the `Problem Statement` was to crowdsource information and let it be easily accessible.

Along with building the ideas conceptually we made sure we had the neccessary technical skills and neccessary technical information to build our ideas. Knowledge of JavaScript was the biggest enabler, allowing us to quickly spurn up an Android Application in a matter of days.

## Our Solution

We identified the two divisions of the problem statement and requirements to be Disaster Education and Disaster Relief.

We named it **CalamiTeacher**.

### Disaster Education

We took a gamified approach towards building the disaster education platform, the ideology was to build `narratives` out of preventive measures to be taken during a disasters, to take the players on a ride with paths they could choose for themselves. A choose your adventure for teaching people about disasters. `It's an Earthquake! What will you do?! Hide under the table or Sit on the Sofa?` (well, the PoC didn't quite end up as our imaginations, welp welp.)

The conceptual inspiration was taken from video games like `Life is Strange`, `Wolfs Among Us` and other choose your own adventure games.
The idea was to build a narrative for a person to follow, allowing them to choose the course of the narrative, which was the event of a disaster happening, penalizing them with providing them the correct information and the consequences which could have led to loss of life and property.

**Example:** 

`If you are stuck in an earthquake, Will you prefer to sit (and sip wine) on a sofa or hide under that hard wooden table. Yes, The Table.` 

Building with this example, we decided to make our PoC as a questionnaire-sort-of experience asking questions and giving binary options to choose from, one leading to safety and the other leading to, well, `loss`. 

<img src='https://i.imgur.com/P03Q9Or.jpg' width='25%' alt='CalamiTeacher'> <img src='https://i.imgur.com/fqrpQ2n.png' width='25%' alt='Choices'>

## Disaster Relief Hepline

For the second part, We were to essentially build a system to provide relief when a calamity hits or when a person was under duress. Right from the beginning of the idea, we decided we needed to keep tech on the person's phone to a minimum because getting internet connectivity during a disaster will always end up as a problem, so we relied on an assumption that 2G GSM networks were available and SMSes could be sent. Thinking a bit more practically, assuming even 2G GSM networks was a long shot during a disaster, but some kind of networking is needed to provide relief, that is why we ended up using 2G GSM as our constraint.   

The concept was to have a service which could receieve conversational messages from a person in duress and provide relief information like nearest police stations, hospitals, relief camps, if any. The API to be built would also allow for people to add information to the database, thus, crowdsource information, and the information could be accessed realtime.

**`A Simplified Chatbot for relief during disasters.`**

**Example:**

A person sends a message to our service,

`I am stuck in a flood at City Centre Mall, Raipur`

The service recieves the message identifies the important intents from the message

`I am` **`stuck in a flood`** `at` **`City Centre Mall, Raipur`**

With this we know, what has happened to the person and where is the person, using this information we can pin point them and find the nearest helplines from that person.

A database of Hospitals, Police Stations in Raipur was built using lists by the government available on the internet and some python data processing magic. This information was stored in the database accessed by the application server powering the service. 

An API was built to access the service, but our main aim was to power it over SMS, Twillio offered programmatic SMS services, the same application server was modified to allow for receving messages from twillio and sending responses back. In the end, We found out Twillio didn't really work with Indian numbers, though the same could be replicated by using the GupShup API for Indian Numbers.

<img src='https://i.imgur.com/nXkKpsZ.png' width='25%' alt='Choices'>

# Technology

The crux of the Hackathon was to build an `Android Aplication` for the SKY phones, With no one in the team having experience with building Native Android Applications, we resorted to using `React-Native` to build our application as the application wasn't very complex in nature requiring any special hardware capabilities we could have wanted from writing it in Java.

To power our relief helpline, A REST API was built with Django and hosted on Heroku.

React Native is a framework for developing native Android and iOS applications in JavaScript.

The application was divided in two swipeable views for dealing with the two parts of the problem.

## Disaster Education

The window was divided in three rows
 - Media for the current question
 - The current question's text
 - Options for the current question
 
(as can be seen in screenshots above)

The idea of a narrative was not implemented to the extent of the concept, it was limited to providing the user correct information when a wrong option was selected and allowing them to select the correct information and proceed to the next question.

Building this section of the application was fairly straightfoward as the data for the questions was taken from a list of question written in a specific format and displayed succesively.

The format for defining a question was decided as (JSON):

```
{   
  'question': `Garbage bins, spare pipes, loose bricks outside house`,
  'img': Images.Questions[0],
  'options': {
      'choices': [ 
          {
              'name': 'Leave them there',                },
          {
              'name': 'Clean them asap', 
          }
      ],
      'correct' : 1,
      'correctText': `Keep the surroundings clean, Don’t let loose objects lie around`,
      'incorrectText': `Don’t let loose objects lie around before occurrence of a cyclone`
  },
}
```

With this structure, all the information about the question could be represented as JSON and stored in a list of question which was rendered by the App.

Source code for the application is available on [GitHub](https://github.com/arush15june/calalmiteacher).

##  Disaster Relief

The Disaster Relief system required NLP for processing messages received by the application, we used DialogFlow for this purpose, as it was easy to build intents and the service exposed a REST API to access the platform.

The application server was built in Django, It received requests from the application and SMS WebHook's and generated Hepline Responses.

DialogFlow was trained to pick up two intents, The Location of a person and the problem they were facing, It worked pretty good,even picking up locations that were not part of the training process most of the times.

When a request, a message, was recieved by the server, the message was sent to DialogFlow to extract the important information from the message. With this information we could recommend helplines nearest to the message sender.

For suggesting nearest helplines, we needed two points of information, coordinates of the sender, coordinates of helplines in our database. Then the euclidean distance of the coordinates would provide the information of the nearest helplines.

### Building the database

The information we extracted from publicly available lists was put in spreadsheets, most of the rows contained information like the Name of the Helpline, the Address, the City, and the Contact Information. To power our API, we needed the Coordinates of these helplines, Google's Geocoding API allowed us to do that. We built a simple Geocoder module and Geocoded the addresses to Coordinates. 

```python
class Geocoder():
    
    API_URL = 'https://maps.googleapis.com/maps/api/geocode/json'
    API_KEY = ''
    
    def __init__(self, location, *args, **kwargs):
        self.location = location
        self.lat = None
        self.long = None
        self._geocode_location()
            
    def _geocode_request(self, address):
        params = {
            'address': address,
            'key': self.API_KEY
        }
        r = requests.get(self.API_URL, params=params, headers={'Cache-Control': 'no-cache'})   
        return r

    def _geocode_location(self):
        geocode = self._geocode_request(self.location)
        
        geocode_data = geocode.json()
        lat = geocode_data['results'][0]['geometry']['location']['lat']
        lng = geocode_data['results'][0]['geometry']['location']['lng']

        self.lat = lat
        self.lng = lng
        
    @property
    def coordinates(self):
        return (self.lat, self.lng)
```

(well, the geocoding api can be a hit or a miss, so multiple tries with the same request might return different results)

With this we had CSVs containing information of Relief Heplines in Chhattisgarh such as Hospitals, NGOs and Police Stations. This data was added to a SQL database with models defined in Django and served over the REST API.

We built a database of approximately 450 Relief Helplines.

# Pitching our application and the event.

Our proposal was selected and we flew to Raipur to pitch our application! The event took place at 36Inc, Raipur, a newly innaugrated Incubator in raipur incubating a lot of interesting startups.

For the competition, we ended up in a round table pitching our application to a panel of people from different domains and not just tech.

# Conclusion

Well, we didn't win the Rs.2 Lakh prize, but we got 3 SKY phones as a token of participation. 
I ended up learning to build an application in React Native and gathered more knowledge on building REST APIs and SMS Powered Apps.

The city of Raipur also had great air compared to Delhi. 10/10 will go again.
