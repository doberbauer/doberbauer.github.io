Title: The joys of self-hosting a wiki for your home
Date: 2025-07-02 10:44
Category: Musings
Tags: dokuwiki, wiki 
Status: draft

*I outline why I chose to self-host a 'House Wiki' and how it's set up*

Owning a house can be a daunting thing. As a home-owner if something breaks in my home I'm responsible for fixing it. There's no landlord or superintendent that I can call - it's on me. And there are so many systems and appliances that can break. 

This kind of thing really brings out my data-hoarder / obsessive organizer side and so I started a wiki to keep track of it all. Any tidbit of information about my house gets filed neatly into the wiki. 

My home wiki is organized by category or as namespaces in DokuWiki parlance. 
There are two groups of categories, one that's available to houseguests and one that's not. 

Available to houseguests 
- House-sitter Guide 
- Appliances 
- Entertainment
- Pets
- Plants
- Recipes 
- Rooms
Not available to houseguests 
- Electrical 
- Garage
- Projects 
- Utilities 

The Home Page for the wiki includes quick links for emergency services, locations of gas shutoff, main water valve and the breaker box, and the information for our animal's vet.

The Appliances page lists appliances by what room they're in, unless they move around a lot in which case they're put in a Misc heading. Each appliance has its own page under the Appliances namespace. I use the [Template Page Name](https://dokuwiki.org/plugin:templatepagename) plugin and set up a template page specific to the Appliances namespace so that whenever I add an Appliance I get a copy of the template with areas for Category, Manufacturer, Model, Manual PDF or Link, Location and Power Specs.
I also added an area for cleaning instructions if nothing is mentioned in the user manual. 

The Entertainment category has information about the streaming services, book and games both board and video we have available. This includes how to use the RecalBox retro game emulator I have set up on one of my Raspberry Pis, and where the PlayStation game discs are kept.

The Pets section has entries for our four animals. I used the Template Page Names plugin here again to standardize headers across our pets. Each pet page has headers for Food, Medicine, Routine Care, Emergency Care, Cleaning and Misc.

The Plants category has information about our plants like what kind of plant it is, how often to water it and so on. 

The Recipes category is where I store recipes we like to make or want to try. I used Template Page Names again here to standardize properties and headings across all recipes. I also created pages specific to each food category and leveraged the Backlinks plugin to view all the recipes within the namespace that link to this category. 

[Just the Recipe](justtherecipe.com) is a helpful tool here. If you find a recipe you like, this tool will strip away all the life story SEO garbage and return only the ingredients and directions.

The Rooms category has a main page where room page links are grouped by First Floor, Second Floor, Outdoor or Utility space. Rooms have their own pages and I used Template Page Names again to standardize headers, as well as Backlinks to refer to pages directed to this from from Appliances, Electrical, Plants, etc. 

The Electrical category contains information about electrical breakers, circuits, outlets, and switches. Rooms are assigned room codes like K for Kitchen, GA for Garage, GU for guestroom and so on.  There are also codes for outlets (O), switches (S) and plates (P). Each outlet, switch and plate is assigned a unique ID by combining the room code, the item code and a numeric index. Ex GA-O-1 is an outlet in the Garage. 
Each room has its own electrical page listing the outlets, plates and switches present in that room as well as which circuit/breaker covers that room. 

Each circuit/breaker has its own page as well. 

With this information, combined with the Power Specs from Appliances, it is easy to sum up the total maximum wattage of all appliances connected to a switch or circuit and determine if it is within its budget. 

The Garage page contains information about our vehicles including license plate number, VIN, year, make, model, mileage, maintenance history, and links to driver handbooks, repair manuals and warranty information. 

The Utilities page contains information for each of our utilities providers including phone number, website and billing frequency.