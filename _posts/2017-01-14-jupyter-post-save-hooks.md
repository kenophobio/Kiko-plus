---
published: false
author: min
title: 'Jupyter Post-Save Hooks: Save Notebook as .html and .py'
tags: 
    - python
    - jupyter
---
**TL;DR**: _I describe how I added a post-save hook for all of my Jupyter notebooks so that both an `.html` and a `.py` version are stored automatically at every save._

![Jupyter logo]({{site.baseurl}}/images/jupyter-main-logo.png)


Finding the perfect collaborative workflow in a data science team is not easy, especially if everybody has their own preferred programming language, mode of working, coding style etc. Even though I am relatively new to the Jupyter world, I can see a lot of potential for it in our team. Hence, you, dear reader, can expect a couple of Jupyter posts coming up that describe some of the small things that I discover while shaping it to my and my team's workflow. 

<!--[^1]: I found [this video](https://www.youtube.com/watch?v=JI1HWUAyJHE) quite useful for a beginner to Jupyter, who is interested in using it in a data science team.
-->

As you might already know Jupyter notebooks are stored as JSON files. This means they are kind of human-readable, but not really, especially not if you do a `git diff`. In addition, I've found myself wanting to take a look at one of my notebooks very quickly (e.g. for copying some code) but having to first start my local Jupyter server so that I can get to the pretty notebook form. There are probably some sophisticated workarounds for these issues (e.g. [this tool](https://github.com/jupyter/nbdime) for getting better diffs), but after watching [this video](https://www.youtube.com/watch?v=JI1HWUAyJHE) I decided to try what was suggested there: Making sure that all my notebooks are also saved (and versioned) in their `.html` and `.py` version.[^1]


[^1]: Saving and versioning the `.html` file may be unconventional as it is clearly derived content. However, I can see scenarios where stakeholders may ask for an old analysis and it should be very convenient to just share the versioned HTML file without having to start a Jupyter server and figuring out that your code doesn't work anymore because some library's API changed (or something equally annoying). 


Now of course we don't want to have to do this manually every time we save our notebooks. Instead, these two steps (i.e. saving as `.html` and saving as `.py`) can be fully automated with the so-called **[post-save hook](http://jupyter-notebook.readthedocs.io/en/latest/extending/savehooks.html)**. With the post-save hook we can define functions which are always called whenever a file is saved. (There is also the pre-save hook which we won't use in this post.)

The snippet of code that I had to add to my `~/.jupyter/jupyter_notebook_config.py` for this purpose is given below (copied from [this gist](https://gist.github.com/jbwhit/881bdeeaae3e4128947c)):

```python
import os
from subprocess import check_call

def post_save(model, os_path, contents_manager):
    """post-save hook for converting notebooks to .py and .html files."""
    if model['type'] != 'notebook':
        return # only do this for notebooks
    d, fname = os.path.split(os_path)
    check_call(['jupyter', 'nbconvert', '--to', 'script', fname], cwd=d)
    check_call(['jupyter', 'nbconvert', '--to', 'html', fname], cwd=d)

c.FileContentsManager.post_save_hook = post_save
```

The official Jupyter documentation actually includes a [similar example](http://jupyter-notebook.readthedocs.io/en/latest/extending/savehooks.html) without calling `jupyter` as a subprocess. Their version is probably faster, but the code is a little less readable. 

With the new `.html` version sharing should be easier than ever. And diffing the `.py` version of the notebook should be way more informative!

