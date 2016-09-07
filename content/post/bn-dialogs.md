+++
date = "2013-04-12T00:00:00+01:00"
draft = true
title = "Bits & Niggles - Dialogs"
image = "bn-dlg/title.jpg"
+++

**UPDATE 2016:
I really had to bring this post across to the new site because it amazes me that to this day there are popular apps that still haven't figured this out.
After we have spent years getting used to and developing applications that fit this paradigm of positive-on-the-right, I _still_ have apps I rely on installed on my
Nougat (Android 7.0) toting Nexus 6 that not only look hideous on its 1440x2560, 492dpi screen, their dialogs still have positive-selection buttons on the left.  
Some even still use Holo!**

Dialogs are a necessary part of Android development, prompting the user for a Yes/No action is often required by applications of various types.

The Yes/No answer to these dialogs is usually of some importance, for example;

> “Are you sure you want to erase every single photograph you have ever taken?”

and the Answer can often be **“No”**, because you clicked on something accidentally or because the UI design didn’t make it entirely clear what action you were about to perform, but that’s a story for another day.

Fortunately, I don’t have many horror stories, most of my precious information is backed up, because that’s good practice, but there have been occasions where I have erased entire configurations and had to re-input a lot of non-mobile-friendly data and strong passwords all over again.

Why? Because the **“OK”** and the **“Cancel”** buttons were not the way round that I expected them to be. Time for a bit of background;

In the days of Gingerbread and earlier, the “positive” action of a dialog belonged on the left, and the negative on the right, great, no problem. 
However, once Ice Cream Sandwich came to phones, this changed. The “correct” way, from now on, is to have the positive button on the right.

The main problem here is that certain developers have hard-coded their buttons a particular way around, or just used a different type of dialog, and even now, far, far down the line, on Jelly Bean 4.2.2 I find even some of the most polished, high quality apps with an excellent attention to detail, still have their positive button on the left hand side when run on a Jelly Bean device.

The reasons this bugs me are twofold, first, I’m used to it being on the right, I have used my phone that way for some time now, and it’s the way I expect it to behave. Worse still, I’ve been using it that long, that when I perform an action, I rarely read the contents of a dialog, and I expect the button I want to be where I want it, and then I click…. oops.

Second, it’s particularly easy to avoid this problem, using a Dialog Builder in Android ensures that you as a developer need not concern yourself with the platform, or where it wants to put its positive and negative buttons. Just set which is positive, and which is negative, and the OS will take care of the rest.

So lets take a look at this horrendously complex code, so complex, that even the greatest of applications out there couldn’t find the time to do right;

``` java
Builder b = new AlertDialog.Builder(this);
 
b.setTitle("Test");
b.setMessage("This is an AlertDialog");
b.setPositiveButton("Ok", new AlertDialog.OnClickListener()
{
    @Override
    public void onClick(DialogInterface dialog, int which)
    {
        // Handle your positive button here
    }
});
b.setNegativeButton("Cancel", new OnClickListener()
{
    @Override
    public void onClick(DialogInterface dialog, int which)
    {
        // Handle your negative button here
    }
});

AlertDialog d = b.create();
d.show();
```

And here is the resulting Dialog on Gingerbread 2.3.3;

![OK/Cancel Dialog on Gingerbread, OK on th left.](/images/bn-dlg/dlg01.png)

We have the positive action on the left, as is expected. So, what happens if we run this on Jelly Bean or ICS?

![OK/Cancel Dialog on Jelly Bean or ICS, OK on th right.](/images/bn-dlg/dlg02.png)

Magic! We now have our positive action on the right hand side, and this all came from exactly the same code!

Job done.