Title: Resources - How I Organize My Resources
Date: 2025-07-02 09:05
Category: Resources
Tags: resources, musings

*How I organize the cool stuff I find on the internet*

For as long as I've been using the internet, when I've come upon an interesting page I've bookmarked it. As my collection grew I started filing bookmarks into folders and subfolders and eventually things started to get a little cumbersome. When I discovered [HackerNews](https://news.ycominator.com) the number of bookmarks I collected per week exploded and my organization approach collapsed. That is, until I learned about Personal Knowledge Management systems like Obsidian. This post is about how I use Obsidian to catalog the useful, cool, often hidden gems of the net. 

When I find a cool thing I bookmark it and file that bookmark into a folder marked "Queue" on my bookmarks toolbar. When I get enough bookmarks I open up my "Resources" Obsidian Vault (notebook in Obsidian parlance) and create a new note for each bookmark. 

# Templates
One powerful feature of [Obsidian](https://obsidian.md/) is Templates. Suppose you're journaling in Obsidian and you want your journal entries to always have the same sections. You could create a Journal Entry template and whenever you want to make an entry you'd simple summon the template and now you have all your sections at the click of a button. When I create a Resource entry I create a new note and use my Resource Template to copy over all the sections that I want. 

The next few sections describe what's in this template.

# Properties
Obsidian allows you to create and assign "Properties" for notes. If you're saving recipes then you might have Properties like `Meal`, `Servings`, and `Calories`. If you're doing project management you might have Properties like `Status`, `Start Date`, `End Date`, `Assignee`. My Resources are much simpler and have `tags`, and `url`. I also have a Description area but it's not a Property.

The `tags` Property lets me file a Resource into different categories without having to put them in different folders. This was one of the drawbacks of my earlier bookmark filing system - bookmarks can belong to multiple categories. Since Obsidian treats whatever I put in `tags` the same as `#hashtags`, I can also pull up the Tags View and see a list of all articles that have a particular tag. 

Properties really shine because they're able to be queried in Dataview.

# Dataview
[Dataview](https://github.com/blacksmithgu/obsidian-dataview) is an Obsidian plugin that lets you...

> Treat your Obsidian Vault as a database which you can query from. Provides a JavaScript API and pipeline-based query language for filtering, sorting, and extracting data from Markdown pages.
>  - *Dataview Github README*

I use Dataview in two places. The first is my Queries folder where I have a set of notes that define Dataview queries on topics that I frequently revisit. For example I have a "Game Dev" query that looks for notes tagged "game-dev", "3d-modeling", "textures", and "animation". The query itself is short and simple:
``` Dataview
LIST
WHERE contains(file.tags,"game-dev") OR contains(file.tags,"3d-modeling") OR contains(file.tags,"textures") OR contains(file.tags,"animation")
```

The other place I use Dataview queries is in the Resource Template. I wanted a See Also section that looked for notes with similar tags. That query looks like this:
```dataview
list
from "Resources"
where any(contains(file.tags,this.file.tags))
sort file.mtime desc
limit 7
```

It looks through the Resources folder and lists the first 7 files, sorted in descending order by the file's modified date, where that file's tags contains this files tags. 

# Graph View

I don't utilize this feature often but the cool factor is unbeatable.
Obsidian's Graph View allows you to visualize notes as nodes in a [force-directed graph](https://en.wikipedia.org/wiki/Force-directed_graph_drawing) where nodes are linked if they reference each other. In this case notes (gray) are linked to their tags (green). The clusters that form are largely due to notes having only a few tags. The more tags a note has the less it will gravitate to one particular tag and be positioned between them. 
![Resources Graph View](https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2Fdrive.google.com/uc?id=1Cbs3IEXvtpi1KyusybyrdJXDIjZxSNjK)
