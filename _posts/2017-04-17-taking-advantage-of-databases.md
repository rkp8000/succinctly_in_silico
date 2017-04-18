---
layout: post
title: "taking advantage of databases"
description: "an introduction to relational databases"
comments: true
---

<style>
table, th, td {
    border-collapse: collapse;
    border: 1px solid black;
}
</style>

So far we've focused mostly on the code side of scientific analysis, but sooner or later you're probably going to start working with data. Since data can wax overwhelming quickly, especially in the present age of high-throughput experiments and dense sensor arrays, it's important to think through how to manage it and interact with it from your codebase from the get-go.

### scientific data

First, what kind of data are we talking about? The answer is very broad, of course. Data might be collected from surveys, clinical trials, animal experiments, physics experiments, computational simulations—the list goes on. So by data we basically mean any structured collection of measurements that we don't want to repeat every time we use it. Obviously it's not too feasible to repeat a two-year clinical trial every time we change a parameter of how we're analyzing it, but this applies to things like computations too. If it takes five minutes to run a complicated regression and generate a plot, it might be good to store some intermediary results so that we can change the plot, which presumably takes milliseconds, without having to redo the regression each time.

### the 90% use case of scientific data storage

If our project is economical enough that we really only expect to need to save one or two things throughout its whole lifespan, we can probably get away with a one-off sort of solution, like saving some intermediary result in a .pickle or .npy file, in one part of our code (e.g., the analysis part) and loading it from another part (e.g., the plotting part). However, at least in my experience, even when I've much expected a project to remain small, the number of results I've found myself wanting to save for later has usually grown out of control, such that if I try to scatter them throughout a bunch of .pickle files I quickly find myself staring a disorganized mess. Consequently I believe that having a principled way of saving results for later use is well worth its while. This not only limits headaches but also makes our project's final results easier to verify and reproduce, be it for the benefit of the scientific community or just for our own sanity.

Here we're going to focus on the 90% use case of scientific projects. That is, even though "big data" is one of those popular buzz words out there in the media, many labs realistically collect a relatively small amount of data, i.e., in the mega- to giga- to terabytes range. This data can of course come in a wide range of forms and formats, so there's not going to be any one-size-fits-all solution to precisely how to curate it; however, there is still one very powerful tool that will help guide us, and which has been extensively studied and optimized in software engineering and commercial software development, but which still (at least in my experience) has a relatively timid presence in science. This is the database.

For the rest of this post we'll talk about what databases are, how they're structured, why they're useful, and how one might look in a toy project. In the next post we'll talk about how to install one and interact with it through Python, and after that we'll talk through some principles of optimizing their design for use in scientific projects.

### the relational database

While there are many types of database out there, the one that has best stood the test of time is the *relational database*, and this is going to be our focus. While there are other databases designed to solve specific problems, the relational database is quite general, and I've never come across a sufficiently sized project that wouldn't benefit from at least some integration of a relational database into its workings.

So what is a relational database? At its most basic, a relational database is a set of fancy spreadsheets. Instead of spreadsheets, however, they're called tables. Each table is defined by its columns, or fields, and each data element takes the form of a row. Say, for instance, we were doing an experiment in which we were showing images of different shapes to people on a screen and recording some physiological response. We might store the data in two tables. First, we might have a table called "trial" in the database that looked something like this:

|id|subject_id|date|trial_block|shape|x_position|y_position|size|rotation|color|mean_response|full_data_file|
|:-|:--------:|:--:|:---------:|:---:|:--------:|:--------:|:--:|:------:|:---:|:-----------:| ------------:|
|1|1|2017-01-02|1|square|3.6|7.1|3.3|45|(0, 1, 0)|12.5|PJ/20170102/1/110521_full.csv|
|2|3|2017-01-03|1|hexagon|1.3|4.2|6.1|90|(1, 1, 0)|19.1|JJ/20170103/1/121025_full.csv|
|3|3|2017-01-03|1|pentagon|3.3|1.1|5.5|45|(1, 0, 1)|39.4|JJ/20170103/1/125622_full.csv|
|4|3|2017-01-04|2|triangle|2.3|1.5|1.1|0|(0, 1, 1)|15.4|JJ/20170104/2/093131_full.csv|
|5|2|2017-01-05|1|square|9.2|1.3|3.7|45|(1, 0, 0)|18.7|AM/20170105/1/094101_full.csv|
|6|2|2017-01-05|1|pentagon|2.5|2.1|9.3|180|(0, 1, 1)|34.8|AM/20170105/1/095758_full.csv|
|7|2|2017-01-05|2|square|6.2|0.3|3.7|90|(1, 0, 1)|12.7|AM/20170105/2/102115_full.csv|
|8|2|2017-01-05|2|pentagon|3.5|3.1|18|180|(0, 1, 1)|39.1|AM/20170105/2/112278_full.csv|
|9|2|2017-01-06|3|square|6.2|0.3|3.7|90|(1, 0, 1)|12.7|AM/20170106/3/092751_full.csv|
|10|2|2017-01-06|3|pentagon|3.5|3.1|18|180|(0, 1, 1)|39.1|AM/20170106/3/102955_full.csv|
|11|5|2017-01-07|1|triangle|3.8|0.9|4.4|90|(0, 1, 0)|21.1|NH/20170107/1/100218_full.csv|
|12|5|2017-01-07|2|pentagon|2.5|3.1|1.2|180|(0, 1, 0)|35.2|NH/20170107/2/110312_full.csv|


As you can see, the table is defined by a set of fields that are listed at the top of each columns, and a set of rows, each of which contains a value for each column. We might also have a table called "subject", that would look something like this:

|id|name|code|age|height|
|:-|:--:|:--:|:-:|:----:|
|1|Pinkus Johanssen|PJ|46|68|
|2|Artemis Mueller|AM|32|60|
|3|Jackie Jones|JJ|26|61|
|4|Nora Hubermeister|NH|23|59|
|5|Elmer Edgerton|EE|24|65|

What makes the tables *fancy* is that they can connect to each other via relationships. In the above, for example, the "subject_id" field of the "trial" table relates (i.e., refers) to the "id" field of the "subject" table. Having such relations greatly simplifies queries one makes to the database. Note also that not every bit of data needs to be directly stored in the rows and columns of the relational database. The full data files, for instance, are stored as external .csv files. However, as we will show in a minute, it is quite nice to have at least a good portion of one’s data stored in the database itself.

The database as a whole, by the way, is typically stored as a file or a small collection of files on one's computer, and there are many graphical viewers that allow you to browse the tables and edit them, as well as do simple operations on them without having to write any code. This is an important advantage, since it's quite useful to have an easy graphical method of looking at the data a scientific project produces.

### a comparison to more naive data management systems

Now, the more you work with relational databases the more you'll start to feel like they're kind of an obvious way to represent data, but I want to explicitly point out their benefits in comparison to alternative methods that also might have felt "obvious" at one point, in particular, storing all of our data in a hierarchy of folders.

To see this, let's say we have a directory organization where the top level directories correspond to subject codes, and within each of these is a short file containing subject information, e.g., their name, age, and height, as well as one folder for each day the subject participated in the experiment. Finally, within each of these folders is a folder for each trial block, and in this lowest level folder there is a text file containing the information for all the trials in that trial block, as well as the full data files for trials in that trial block. It might look something like this:

```
|-- experiment
    |-- AM
        |-- subject_info.txt
        |-- 20170105
            |-- 1
                |-- trials.txt
                |-- 094101_full.csv
                |-- 095758_full.csv
            |-- 2
                |-- trials.txt
                |-- 102115_full.csv
                |-- 112278_full.csv
        |-- 20170106
            |-- 3
                |-- trials.txt
                |-- 092751_full.csv
                |-- 102955_full.csv
    |-- EE
        |-- subject_info.txt
        |-- 20170107
            |-- 1
                |-- trials.txt
                |-- 100218_full.csv
            |-- 2
                |-- trials.txt
                |-- 110312_full.csv
    |-- JJ
        ...
    |-- NH
        ...
    |-- PJ
        ...
```

While at first glance this might also seem like a reasonable way to organize a dataset, since the data we're storing does feel like it has a sort of natural hierarchy to it (e.g., trials per day, days per subject, subjects per experiment), let's look at how we'd actually query it. For instance, suppose we want to get the names of all the subjects that were shown a specific shape at some point during the experiment. The code would look something like this:

```
data_root = ...
shape = ...
names = []

# loop through subject directories
subject_dirs = [sd for sd in os.listdir(data_root) if os.path.isdir(sd)]

for subject_dir in subject_dirs:
    # open 'subject_info.txt' and get name
    with open(os.path.join(subject_dir, 'subject_info.txt'), 'rb') as f:
        ...
        name = ...

    # assume subject was not shown shape
    has_shape = False

    # loop through date and trial block directories
    date_dirs = [dd for dd in os.listdir(subject_dir) if os.path.isdir(dd)]
    for date_dir in date_dirs:
        trial_block_dirs = [tbd for tbd in os.listdir(date_dir) if os.path.isdir(tbd)]
        for trial_block_dir in trial_block_dirs:
            # open trials.txt and extract shape
            with open(os.path.join(trial_block_dir, 'trials.txt'), 'rb') as f:
                ...
                trial_block_shapes = ...
            # check if shape was shown
            if shape in trial_block_shapes:
                names.append(name)
                has_shape = True
                break

        if has_shape:
            break
```

As we can see, this simple query, which could be stated pretty straightforwardly in English, required a whole lot of code. We had to use multiple nested loops and a flag to indicate if the subject had seen the shape or not, and we didn't even include the code to read the .txt files. In other words, the code is messy, a bit challenging to understand, and introduces many points of possible failure, should there be a bug in it somewhere. (If we stored our data in a large nested dictionary, by the way, it would be similarly complicated to query, since we'd basically still have to loop over a lot of different levels.)

### querying from a relational database with Python

Now let's look at how we'd extract the same information from the example tables shown above. The first snippet is in [SQL](https://en.m.wikipedia.org/wiki/SQL), (pronounced "sequel"),  which may be familiar to those of you with some experience with databases. In this blog we won't talk explicitly about SQL much since we rarely need it, but I thought I'd show what it looks like here, since it's the hidden language that actually underlies the database queries. The second snippet is in Python, using the library sqlalchemy. You don't need to worry about the Python syntax here, however, since we'll unpack it in the next post, but hopefully it's be more or less comprehensible.

SQL:

```
SELECT DISTINCT name FROM subject JOIN trial WHERE shape = @shape;
```

Python:

```
names = session.query(Subject.name).join(Trial).filter(
    Trial.shape == shape).distinct()
```

The key takeaway here is that the code that queries the relational database is immensely simpler than the previous code block that combed through the directory hierarchy. This is because the relational database flattens the data as much as possible, enabling a much simpler interaction. It's less complicated, easier to understand, and there are far fewer points of possible failure.

Here's how we might perform a couple of other queries:

Select the shapes presented for all the trials where the mean response was between 30 and 40:

```
    shapes = session.query(Trial.shape).filter(
        Trial.mean_response.between(30, 40))
```

Select all trials from the first trial blocks of all subjects:

```
    trials = session.query(Trial).filter(Trial.trial_block == 1)
```

Select the mean responses for all AM's trials in which the rotation was 90 degrees:

```
    mean_responses = session.query(Trial.mean_response).join(Subject).filter(
        Subject.code == 'AM', Trial.rotation == 90)
```

If you were to try writing these queries in the context of the directory hierarchy you would almost certainly find yourself quickly overwhelmed, and you would probably have to do a lot more testing to ensure that you weren't making any mistakes. Further, for a real a scientific project we'd likely have many more tables, each corresponding to a class of "units" in the project, and keeping track of them without a relational database would quickly become a big mess.

The goal of this post was to introduce the concept of a relational database and convince you that they can generally make life a whole lot easier. Hopefully you’ve found it informative and are maybe even starting to consider integrating a database into your project if you haven't already. In the next post we'll talk about how to install a database called PostgreSQL and how to interact with it through Python, and after that we'll talk about some principles for optimizing the way we integrate a database into a scientific project.

Until next time.