# web-scraper

This is based on a tutorial with this link
https://morioh.com/p/4455499349ea?fbclid=IwAR1sIvrvE2rP7U4s2QnGv6eMMZGEOq2lhCgNrxrArhlEcTFLhfY8aLZLZkM


Ella Windler is the writer of this tutorial

3 days ago

Open options
How to Build A Web Scraper with Node.js
Web scraping refers to the process of gathering information from a website through automated scripts. This eases the process of gathering large amounts of data from websites where no official API has been defined.

The process of web scraping can be broken down into two main steps:

Fetching the HTML source code of the website through an HTTP request or by using a headless browser.
Parsing the raw data to extract just the information you’re interested in.
We’ll examine both steps during the course of this tutorial. At the end of it all, you should be able to build a web scraper for any website with ease.

Prerequisites
To complete this tutorial, you need to have Node.js (version 8.x or later) and npm installed on your computer. This page contains instructions on how on how to install or upgrade your Node installation to the latest version.

Getting started
Create a new scraper directory for this tutorial and initialize it with a package.json file by running npm init -y from the project root.

Next, install the dependencies that we’ll be needing too build up the web scraper:

    npm install axios cheerio puppeteer --save
Here’s what each one does:

Axios: Promise-based HTTP client for Node.js and the browser
Cheerio: jQuery implementation for Node.js. Cheerio makes it easy to select, edit, and view DOM elements.
Puppeteer: A Node.js library for controlling Google Chrome or Chromium.
You may need to wait a bit for the installation to complete as the puppeteer package needs to download Chromium as well.

Scrap a static website with Axios and Cheerio
To demonstrate how you can scrape a website using Node.js, we’re going to set up a script to scrape the Premier League website for some player stats. Specifically, we’ll scrape the website for the top 20 goalscorers in Premier League history and organize the data as JSON.

Create a new pl-scraper.js file in the root of your project directory and populate it with the following code:

    // pl-scraper.js
    
    const axios = require('axios');
    
    const url = 'https://www.premierleague.com/stats/top/players/goals?se=-1&cl=-1&iso=-1&po=-1?se=-1';
    
    axios(url)
      .then(response => {
        const html = response.data;
        console.log(html);
      })
      .catch(console.error);
If you run the code with node pl-scraper.js, a long string of HTML will be printed to the console. But how can you parse the HTML for the exact data you need? That’s where Cheerio comes in.

Cheerio allows us to use jQuery methods to parse an HTML string and extract whatever information we want from it. But before you write any code, let’s examine the exact data that we need through the browser dev tools.

Open this link in your browser, and open the dev tools on that page. Use the inspector tool to highlight the body of the table listing the top goalscorers in Premier League history.

node-scraper-1

As you can see the table body has a class of .statsTableContainer. We can select all the rows using cheerio like this: $('.statsTableContainer > tr'). Go ahead and update the pl-scraper.js file to look like this:

    // pl-scraper.js
    
    const axios = require('axios');
    const cheerio = require('cheerio');
    
    const url = 'https://www.premierleague.com/stats/top/players/goals?se=-1&cl=-1&iso=-1&po=-1?se=-1';
    
    axios(url)
      .then(response => {
        const html = response.data;
        const $ = cheerio.load(html);
        const statsTable = $('.statsTableContainer > tr');
        console.log(statsTable.length);
      })
      .catch(console.error);
Unlike jQuery which operates on the browser DOM, you need to pass in the HTML document into Cheerio before we can use it to parse the document with it. After loading the HTML, we select all 20 rows in .statsTableContainer and store a reference to the selection in statsTable. You can run the code with node pl-scraper.js and confirm that the length of statsTable is exactly 20.

node-scraper-2

The next step is to extract the rank, player name, nationality and number of goals from each row. We can achieve that using the following script:

    // pl-scraper.js
    
    const axios = require('axios');
    const cheerio = require('cheerio');
    
    const url = 'https://www.premierleague.com/stats/top/players/goals?se=-1&cl=-1&iso=-1&po=-1?se=-1';
    
    axios(url)
      .then(response => {
        const html = response.data;
        const $ = cheerio.load(html)
        const statsTable = $('.statsTableContainer > tr');
        const topPremierLeagueScorers = [];
    
        statsTable.each(function () {
          const rank = $(this).find('.rank > strong').text();
          const playerName = $(this).find('.playerName > strong').text();
          const nationality = $(this).find('.playerCountry').text();
          const goals = $(this).find('.mainStat').text();
    
          topPremierLeagueScorers.push({
            rank,
            name: playerName,
            nationality,
            goals,
          });
        });
    
        console.log(topPremierLeagueScorers);
      })
      .catch(console.error);
Here, we are looping over the selection of rows and using the find() method to extract the data that we need, organize it and store it in an array. Now, we have an array of JavaScript objects that can be consumed anywhere else.

node-scraper-3

Scrape a dynamic website using Puppeteer
Some websites rely exclusively on JavaScript to load their content, so using an HTTP request library like axios to request the HTML will not work because it will not wait for any JavaScript to execute like a browser would before returning a response.

This is where Puppeteer comes in. It is a library that allows you to control a headless browser from a Node.js script. A perfect use case for this library is scraping pages that require JavaScript execution.

Let’s examine how Puppeteer can help us scrape news headlines from r/news since the newer version of Reddit requires JavaScript to render content on the page.

node-scraper-4

It appears, the headlines are wrapped in an anchor tag that links to the discussion on that headline. Although the class names have been obfuscated, we can select each headline by targeting each h2 inside any anchor tag that links to the discussion page.

Create a new reddit-scraper.js file and add the following code into it:

    // reddit-scraper.js
    
    const cheerio = require('cheerio');
    const puppeteer = require('puppeteer');
    
    const url = 'https://www.reddit.com/r/news/';
    
    puppeteer
      .launch()
      .then(browser => browser.newPage())
      .then(page => {
        return page.goto(url).then(function() {
          return page.content();
        });
      })
      .then(html => {
        const $ = cheerio.load(html);
        const newsHeadlines = [];
        $('a[href*="/r/news/comments"] > h2').each(function() {
          newsHeadlines.push({
            title: $(this).text(),
          });
        });
    
        console.log(newsHeadlines);
      })
      .catch(console.error);
This code launches a puppeteer instance, navigates to the provided URL, and returns the HTML content after all the JavaScript on the page has bee executed. We then use Cheerio as before to parse and extract the desired data from the HTML string.

node-scraper-5

Wrap up
In this tutorial, we learned how to set up web scraping in Node.js. We looked at scraping methods for both static and dynamic websites, so you should have no issues scraping data off of any website you desire.

Original article sourced at: https://pusher.com

#nodejs 

