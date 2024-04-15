---
title: "PDF Searching Programmes"
date: 2019-10-07
---

I am trying to write some programmes to search PDFs from directories.

I am hoping to use:
  - Bash
  - Python
  - Perl
  - Ruby

I have done the first two, and am timing them now (from `~/Desktop/Study/.../2018/Tri\ 2/`, searching for `allport`):

|| Bash | Python 
--- | --- | ---
|| 49.80 seconds | 119.56 seconds
|| 42.87 seconds | 118.58 seconds
|| 43.82 seconds | 124.00 seconds
**Average:** | **45.50 seconds** | **120.71 seconds**

&nbsp;

Note that I did Bash &rarr; Py &rarr; Bash..., not Bash &times; 3 &rarr; Py &times; 3.

On average, my Bash script was 2.65 times faster than my python one.

---

A note from the future: once I figured out the logic required to do such a task as this, this was my first good practice with different languages, learning the syntax of these different languages.

---

# More notes on different languages, from mid-October 2019

I want to write more PDF-searchers, for my own edification more than for their functionality.  Here are some notes on the strengths of different languages, and my method for PDF-searches.

### Why use Perl?
  - Text manipulation and data wrangling is easy and fast!
  - Good for "glue" projects in two disparate systems
  - The most complete language

### Why use Ruby?
  - Automation and scripting
  - Data scraping and general crawling
    - Mechanise, Cucumber, Capybara, Site Prism, Selenium, Faker, Pry, Watir
  - Server management
  - General perpose!  Even AI: game bots; social moderation

### Why use LISP?
  - Most <u>programmable</u>; <i>specialised in general purpose</i>!

### Why use Elixir?
  - Elixir runs on the erlang vm
    - Very self-contained
  - Functional programming

### Why use Rust?
  - In sys-programmung, memory safe!
  - Cargo makes managing crates easy!
  - Could replace C++

### Why use Lua?
  - Lua is faster to develope that C++ and doesn't require compiling!

## How I am writing PDF-searches:
  1. PDF &rarr; text with set file and search string
  2. Search file for search string
  3. Print whether or not found
  4. Make search string arg
  5. Make loop in walk dirs + subdirs
  6. Print all PDFs for which found; count and print times found on x PDFs
  7. Time script
  8. Print help output.
