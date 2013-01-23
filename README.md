ComplimentKit
=============

So you're interested in integrating [Compliment](http://compliment.rentzsch.com/) support. Great!

Sorry to let you down, but there's no .m files here. Nor is there an Xcode project or even a precompiled .framework.

Turns out it's so easy to integrate Compliment that such things are overkill.

All you need to do is send a distributed notification, like so:

    - (void)applicationDidFinishLaunching:(NSNotification*)aNotification {
        NSDictionary *complimentRequestDictionary = @{
            @"name": @"MarkdownLive", // required
            @"url": @"https://github.com/rentzsch/markdownlive", // required
            @"path": [[NSBundle mainBundle] bundlePath], // suggested but optional
            @"icon": [[NSImage imageNamed:NSImageNameApplicationIcon] TIFFRepresentation], // suggested but optional
        };
        [[NSDistributedNotificationCenter defaultCenter] postNotificationName:@"ComplimentRequest"
                                                                       object:[complimentRequestDictionary description]
                                                                     userInfo:nil
                                                                      options:NSNotificationDeliverImmediately];
    }

Copy-and-paste away! The two important bits to get right is your app's name (`MarkdownLive` in the example) and your app's Flattr-registered URL (`https://github.com/rentzsch/markdownlive` in the example). The rest is boilerplate.

Once in place, your app will post a Compliment request every time it's launched. Don't worry too much about requesting Compliments too often -- Flattr only allows flattring a thing once per month (though Compliment keeps request logs, so you don't want to ask for a compliment once per second or anything silly like that).

One point of oddity in the above code is that we avoid the seemingly-fine `userInfo` and instead pass the request dictionary as a serialized ASCII plist. Unfortunately sandboxed apps are disallowed from sending distributed notifications with userInfos, so this is our work-around.

To keep things simple, we completely ignore `userInfo`, so don't use it, even in situations you could.

Subrequests
-----------

Does your app make use of an open-source project? It's easy to have your users automatically Flattr those projects along with your app.

Just add a `subrequests` array to your ComplimentRequest like so:

    - (void)applicationDidFinishLaunching:(NSNotification*)aNotification {
        NSDictionary *complimentRequestDictionary = @{
            @"name": @"MarkdownLive", // required
            @"url": @"https://github.com/rentzsch/markdownlive", // required
            @"path": [[NSBundle mainBundle] bundlePath], // suggested but optional
            @"icon": [[NSImage imageNamed:NSImageNameApplicationIcon] TIFFRepresentation], // suggested but optional
            @"subrequests": @[
                @{
                    @"name": @"Discount",
                    @"url": @"https://github.com/Orc/discount",
                },
                @{
                    @"name": @"Sparkle",
                    @"url": @"https://github.com/andymatuschak/Sparkle"
                },
            ]
        };
        [[NSDistributedNotificationCenter defaultCenter] postNotificationName:@"ComplimentRequest"
                                                                       object:[complimentRequestDictionary description]
                                                                     userInfo:nil
                                                                      options:NSNotificationDeliverImmediately];
    }

Notice you don't supply a path or an icon for subrequests.


Scripting with Python
---------------------

You can also create the ComplimentRequest notifications with a simple Python script.

    from Foundation import *


    data = { 'name': 'MarkdownLive',
    	 'url' : 'https://github.com/rentzsch/markdownlive'
    }
		
    nc = Foundation.NSDistributedNotificationCenter.defaultCenter()

    nc.postNotificationName_object_userInfo_options_(
      "ComplimentRequest", NSDictionary.dictionaryWithDictionary_(data).description(), None,
      NSNotificationDeliverImmediately);
