Title: Random Person Generator
Date: 2024-01-01 12:00
Category: Exploration

The Watch Dogs video game series is known for its innovative hacking mechanics, immersive open world and engaging story. But for me one of the coolest things about the Watch Dogs games is the ambient storytelling of the NPCs. When you scan an NPC a nice UI element pops up revealing some noteworthy stats from their Blume profile. Name, occupation, income, a headshot, and a little blurb that humanizes them. That guy you just wasted wasn't just nameless NPC fodder, he was James McCarthy, age 24, dental assistant. He had $3260 in his bank account and he regularly attends renaissance fairs. Well...attended.  This little bit of info goes a long way toward building an immersive experience. It was also functional in Watch Dogs since you could see how much money an NPC has before draining their bank account.

I wondered...how hard would it be to make a NPC generator like that?

Let's outline some of what we'll want.
- Name
- Age
- Occupation
- Income
- Marital status 
- Phone number 
- Social security number 


### Name
I had a large dataset of 8336 names lying around from a NFT project that didn't get off the ground. That seemed like a good start. It also includes the name's origin (Anglo, Arabic, Spanish, Russian, Korean, Japanese, Indian, and Chinese) and typical gender identity (unused).

### Phone number 

We could just randomly generate 7 digits and an area code but we can do better. I found [this](https://www.wirefly.com/area-codes) handy list of US area codes. We could go one step further and get common prefix / exchange numbers for the state that the area code belongs to, but for now an area code is good enough. We're also going to randomly generate the line / subscriber number. 


### Occupation 
You gotta make a living somehow, so let's give our friendly NPCs some jobs. Finding a list of occupations online is easy. It took a little bit more digging to find a list with salary ranges. Now we just randomly index into the data table and generate a random salary within the salary range. Let there be jobs. 


### Age 
Let's give them an age and while we're at it let's give them a birth date. It'll be easier if we start with the birth date and calculate the difference between today and the birthday to get their age. 

```Python
import datetime
import random
def get_dob():
    end = datetime.date(2004,1,1) #Most recent time
    start = datetime.date(1927,1,1) #Least recent time
    delta = end - start #Difference
    int_delta = (delta.days * 24 * 60 * 60) + delta.seconds 
    random_second = random.randrange(int_delta)
    return start + datetime.timedelta(seconds=random_second)
def calculate_age(born):
    today = datetime.date.today()
    return today.year - born.year - ((today.month, today.day) < (born.month, born.day))
```
Seems needlessly complicated, right? 
Like why is the range from 1927 to 2004? 
In 2023 someone born in 2004 would be 19. Since I'm more focused on adding attributes than intertwining them it's easier to cap all the random people to age 19 so whatever job they get is more or less reasonable. The upper cap on age is 96. Seems a little excessive for someone to be working at 96 but who am I to judge? 
Once we have the date of birth it's trivial to calculate age. 


### Marital status 
Now that we have age we can set marital status. You can't be married if you're under 18. Sorry kids. Otherwise they have an equal shot among the options.
```Python
import random
options = ['Single', 'Married','Separated','Divorced','Widowed']
def get(age):
    if age >=18:
        return random.choice(options)
    else:
        return 'Single'
```

### Personality type
How about personality types? 
There are 16 Meyers Briggs personality types. 

``` Python
import random as r
def get():
    first = ["I", "E"]
    second = ["N", "S"]
    third = ["T", "F"]
    fourth = ["P", "J"]
    pers = r.choice(first) + r.choice(second) + r.choice(third) + r.choice(fourth)
    return pers
```

### Astrological sign 
What the hell, might as well throw in astrological sign too.

``` Python
def get(dob):
    dob = str(dob)
    dob = dob.split('-')
    month = int(dob[1])
    day = int(dob[2])
    astr = ''
    if (month==1 and day>=20) or (month==2 and day<=18):
        astr = 'Aquarius'
    elif (month==2 and day>=19) or (month==3 and day<=20):
        astr = 'Pisces'
    elif (month==3 and day>=21) or (month==4 and day<=19):
        astr = 'Aries'
    elif (month==4 and day>=20) or (month==5 and day<=20):
        astr = 'Taurus'
    elif (month==5 and day>=21) or (month==6 and day<=20):
        astr = 'Gemini'
    elif (month==6 and day>=21) or (month==7 and day<=22):
        astr = 'Cancer'
    elif (month==7 and day>=23) or (month==8 and day<=22):
        astr = 'Leo'
    elif (month==8 and day>=23) or (month==9 and day<=22):
        astr = 'Virgo'
    elif (month==9 and day>=23) or (month==10 and day<=22):
        astr = 'Libra'
    elif (month==10 and day>=23) or (month==11 and day<=21):
        astr = 'Scorpio'
    elif (month==11 and day>=22) or (month==12 and day<=21):
        astr = 'Sagittarius'
    elif (month==12 and day>=22) or (month==1 and day<=19):
        astr = 'Capricorn'
    return astr
```

### Birth place and SSN
Now to figure out where they were born. At the same time we'll get them a social security number. 
I threw all the state data into one tsv sheet for easy parsing. 
```
Line[0] = State name
Line[1] = State abbreviation 
Line[2] = State capital 
Line[3] = SSN 1
Line[4] = SSN 2
```
So all we do is open the tsv file, get a random line and parse it for the info we need. 

SSNs have 3 area numbers, 2 group numbers and 4 serial numbers. 
Each state has a range of area numbers.
We snag the upper and lower SSN numbers  for the person's birth state (zfilling until we get 3 digits). 
Group and serial numbers are totally random. The whole thing is assembled into a string and returned.

``` Python
import random
def get(line):
    s_min = int(line[3])
    s_max = s_min
    if line[4] != 0:
        s_max = int(line[4])
    s = str(random.randint(s_min, s_max+1)).zfill(3)
    g = random.randint(0,100)
    g = str(g).zfill(2)
    n = str(random.randint(0,10000)).zfill(4)
    ssn = s + "-" + g + "-" + n
    return ssn
```

### Physical attributes 
For fun I thought it'd be neat to add physical aspects of characters. Height, weight, eye color, hair color, body mass index and BMI class. 

```Python
import random
eye_color = ["Green","Blue","Brown","Hazel","Amber"]
hair_color = ["Black", "Brown", "Auburn","Red","Blonde", "Gray","White","Pink","Green","Blue","Orange"]
def get(age):
    weight = random.randrange(90,250)
    eye = random.choice(eye_color)
    hair = random.choice(hair_color)
    height1 = random.randrange(4,6)
    height2 = random.randrange(0,12)
    bmi = round((weight/(height1*12)**2)*703,1)
    bmi_class = ''
    if bmi <=18.5:
        bmi_class = 'Underweight'
    elif bmi >=18.5 and bmi <= 24.9:
        bmi_class = 'Healthy'
    elif bmi >=25.0 and bmi <=29.9:
        bmi_class = 'Overweight'
    else:
        bmi_class = 'Obese'
    return weight,eye,hair, (height1,height2),(bmi,bmi_class)
```


### Future additions
It'd be nice to procedurally generate a face based on the data we have so far. 

It'd also be nice to expand out birth places to locations other than the US. 

You can be born somewhere and not live there, so I want to delink those attribute generators. 

I made this before ChatGPT was released, so now that it's available it'd be great to leverage that to generate a little blurb about the person. 

I also thought it might be fun to give them a fingerprint. I found [this](http://biolab.csr.unibo.it/ResearchPages/SFinGe_Download.asp) Italian research site with a synthetic fingerprint generator, but it's not exactly open source and they have no reason to let me play with their tool. I attempted to generate fingerprints in Stable Diffusion but they looked like garbage. I suppose I could engineer the prompts a bit more but I lost patience for it. 

---

Note: I made this tool before ChatGPT burst onto the scene like the Kool Aid man. ChatGPT can generate everything below with a simple prompt.

**Prompt:**
Generate a table of 5 randomly generated people. Include information such as a fake phone number, fake social security number, age, date of birth, name, ethnicity, Meyer's Briggs personality type, astrological sign and a short blurb about noteworthy aspects of their recent life.

**ChatGPT:**
Here's a table of 5 randomly generated people:

| Person | Phone Number     | Social Security Number | Age | Date of Birth | Name              | Ethnicity | MBTI  | Astrological Sign | Recent Life Noteworthy Aspect                                     |
|--------|-----------------|------------------------|-----|---------------|-------------------|-----------|-------|-------------------|------------------------------------------------------------------|
| 1      | (555) 123-4567  | 123-45-6789            | 32  | Jan 15, 1991  | Sarah Johnson     | Caucasian | ENFP  | Capricorn         | Recently started a nonprofit organization focused on youth education. |
| 2      | (555) 987-6543  | 987-65-4321            | 45  | Oct 3, 1978   | Javier Rodriguez  | Hispanic  | ISTJ  | Libra             | Completed a marathon and raised funds for a local charity.          |
| 3      | (555) 246-8135  | 246-81-3579            | 28  | May 6, 1995   | Emily Chen        | Asian     | INFP  | Taurus            | Published a book of poetry and received critical acclaim.           |
| 4      | (555) 369-1472  | 369-14-7258            | 57  | Feb 12, 1966  | Michael Thompson  | African   | ENTJ  | Aquarius          | Won an award for his groundbreaking research in renewable energy.   |
| 5      | (555) 802-4679  | 802-46-7913            | 19  | Aug 22, 2004  | Liam Anderson     | Caucasian | ESFP  | Leo               | Featured in a local art exhibition for his vibrant abstract paintings. |

Please note that the phone numbers, social security numbers, and personal details provided in this table are entirely fictional and do not correspond to real individuals.
