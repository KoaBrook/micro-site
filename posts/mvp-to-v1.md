---
title: "MVP to V1: Creating my portfolio website with React and the Airtable API"
description: I created a personal website while on a business trip back in July 2019. It was thrown together in a couple of days using plain HTML and CSS and a pretty okay visual design. Now that I'm in the job market again and finally looking to jump into development professionally, I wanted to remake my portfolio website with a little more pizazz.
date: 2019-11-24
tags: 
    - webdev
    - react
layout: layouts/post.njk
---
I created a personal website while on a business trip back in July 2019. It was thrown together in a couple of days using plain HTML and CSS and a pretty okay visual design. Now that I'm in the job market again and finally looking to jump into development professionally, I wanted to remake my portfolio website with a little more pizazz.

I had a few requirements for this:
1. I would start with an MVP and build upon it
2. It had to be made in code, not with a website or blog builder
3. It must be modular with the ability to add new projects with as little code as possible
4. The website itself should contain a simple list of my projects


### MVP
With my requirements set, I took to creating an MVP product. Since the website would be a list of my projects, the MVP was also a simple list of my projects publicly available online. I used Airtable for this. [Check out the MVP here](https://airtable.com/shre8qIxVeziaXfOQ/tblMvHba5g75ohHHM?blocks=hide).

🖼️ [Airtable MVP: Click to view image](https://thepracticaldev.s3.amazonaws.com/i/swvnp5986qk00g673zs7.png)

One of the great things about Airtable is how it automatically generates unique API documentation for every sheet and view in the base. This was the perfect springboard into the modular concept for the site, in which I wouldn't need any code to add new portfolio entries.


### React web app
I enjoy coding in React. I find the modular nature of components to be intuitive. I used React previously for [Smashesque.com](smashesque.com) and had a good time, so I went with it again. Bootstrap is my framework of choice for throwing together pretty sites so I chose to use it too.


### Modular lists using Airtable
With the help of [Tania Rascia](https://dev.to/taniarascia)'s article on [Using Context API in React (Hooks and Classes)](https://www.taniarascia.com/using-context-api-in-react/), I used Axios and the Airtable API to grab my view of choice and all the rows, fields and content therein from my MVP Airtable. My implementation is a little messy, but it worked, so no problem!
I started with EntryContexts.js which performs the API call and creates a context state containing the spreadsheet object.

```javascript
    import React, { Component } from 'react'
    import axios from 'axios'
    export const EntryContext = React.createContext()
    class EntryContextProvider extends Component {
        state = {
            entries: []
        }
        componentDidMount() {
            const fetchData = () => {
                axios
                    .get('https://api.airtable.com/v0/appeDXIgWSt9xRB6n/
                    Portfolio%20Entries?api_key=[MY_API_KEY]')
                    .then(({ data }) => {
                        this.setState({
                            entries: data.records
                        })
                    })
                    .catch(console.log)
            }
            fetchData();
        }
        render() {
            return (
                <EntryContext.Provider value={{ ...this.state }}>
                    {this.props.children}
                </EntryContext.Provider>
            )
        }
    }
    export default EntryContextProvider
```

Next I created a component called EntryList.js maps the EntryContextProvider component's state data into some simple HTML elements:
```javascript
import React from 'react'

const ListEntry = props => {
    const EnEntry = props.entryData.map((entry, key) => {


        return (
            <div>
                <h3>{entry.fields.title}</h3>
                <p>{entry.fields.notes}</p>
                <p><a href={entry.fields.link}>Link</a></p>
            </div>
        )

    })
    return <div>{EnEntry}</div>
}

export default ListEntry
```

Finally, I created a page called `Entries.js` which ties the `EntryContextProvider` and `ListEntry` components together and displays them on the page in simple React fashion. In this case, it is displayed as a list of portfolio entries on the home page of the website.
```javascript
import React, { Component } from 'react'
import { EntryContext } from '../contexts/EntryContext'
import ListEntry from '../components/EntryList'

class Entries extends Component {

    render() {
        return (
            <EntryContext.Consumer>{(context) => {
                const { entries } = context
                return (
                            <ListEntry entryData={entries} />
                )

            }}
            </EntryContext.Consumer>
        )
    }
}

export default Entries
```

In App.js, I wrapped my site in the EntryContextProvider component, which ensures that every page has access to the Airtable context.
```javascript
<EntryContextProvider>
    <Switch>
        <Route exact path="/" component={Entries} />
    </Switch>
</EntryContextProvider>
```

Finally, I had the results I wanted! A simple list of all portfolio entries that were in my Airtable spreadsheet:

🖼️ [Simple list: Click to view image](https://thepracticaldev.s3.amazonaws.com/i/cybsyolz0md43i9uq9kr.PNG)


### Aesthetic challenges
Many developers revel with minimal websites with lists of achievements and projects. A white colour scheme and emoji are both very popular. I enjoy being a bit contrarian and a total 90s kid, so I took inspiration from the new [SEGA MegaDrive Mini website](https://megadrivemini.sega.com/) and tried to match its look. Unfortunately, there's a lot of history, imagery and the theme of a retro console that helps bring the 90s Spaceship look together. Without these things (and a lack of artistic talent at my disposal) the results were less than inspiring. I realised that a dark theme for my portfolio was somewhat uninviting and less friendly than I wanted it to be, so I ended up going with a light theme. I wanted to keep some semblence of character, so I kept a scrolling background grid and gave the primary container a "sheet of paper" look. At this point I decided to add images for each project and an emoji to identify what kind of project each is, again all contained in the spreadsheet and called with the Airtable API. I hope the emoji are intuitive to anyone viewing the portfolio but the verdict is still out on that. Once everything was styled, I was extremely happy with the outcome:

🖼️ [Styled list: Click to view image](https://thepracticaldev.s3.amazonaws.com/i/vttiq9v63fwl0p0jwmva.PNG)


### Final Touches
Since my website was made from scratch, I considered it an addition to my portfolio. However, I didn't want it to be added to the list with a link to itself. Therefore I added a ❔ icon in the upper-left which triggered a popover that gives more information on the site. This article will be added onto it, too:

🖼️ [Popover: Click to view image](https://thepracticaldev.s3.amazonaws.com/i/01gy48dmiozz2ke2lbrw.PNG)

Finally, there was a site-breaking bug to be squashed. An empty field in the spreadsheet caused the entire Airtable context to fail, causing a blank web-page. I added some very rudimentary validation to resolve this but I didn't over-think it too much since the airtable should never have empty fields if I'm managing it. At the very least, correct entries load as they should with a simple inline error if there are any problems with a field:

🖼️ [Simple inline errors: Click to view image](https://thepracticaldev.s3.amazonaws.com/i/3fc7ob76vhqp02b08vxp.PNG)

And that's about it for my V1 portfolio website! To add new projects I just add a row to the sheet, avoiding any code at all. Let's look at my requirements from the beginning of the project:

1. I would start with an MVP and build upon it ✔
2. It had to be made in code, not with a website or blog builder ✔
3. It must be modular with the ability to add new projects with as little code as possible ✔
4. The website itself should contain a simple list of my projects ✔

As you can see, I hit all four of my requirements! It was a great journey and an interesting project. I learned the Airtable API, the importance of validation and plenty of design quirks. I'm very happy with the end result!


### What's next?
I enjoy the site as it is and will most likely keep it simple for now. I may use more spreadsheets to add additional list-based sections to the site- articles, testimonials, cat photos... whatever I want to add, I can do so with very little code- Clone the `Entries`, `EntryContextProvider` and `ListEntry` components, replacing the Airtable API link and making any styling changes I want to.
Airtable is not ideal for, say, entire blog posts but I'm actually curious about whether it could be done. Imagine an entire site with an Airtable backend? It's possible and perhaps I'll dabble in that idea in the future. For now, I'm happy to mark this V1 project complete! 


### BONUS 
I just added a new field to the Airtable named "order" and a new code block. With this new snippet, I can adjust the order in which the list entries appear by adding an order value in Airtable!

```javascript
const { entries } = context
    let sortedEntries = entries.sort(
        function(a,b){
            return a.fields.order - b.fields.order
            })
```

[Check out the live site here](https://shemthedev.netlify.com/)
[Check out the Airtable backend (MVP) here](https://airtable.com/shre8qIxVeziaXfOQ/tblMvHba5g75ohHHM?blocks=hide)
[Check out my Github here](https://github.com/ShemTheDev)

I'm currently looking for work! Drop me an email at sgyll@protonmail.com if you'd like to chat.


