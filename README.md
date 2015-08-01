# Bot for Vanilla

Sometimes you want a [minion](https://github.com/vanilla/minion) to do your dirty work, but sometimes you want a _personality_ to make your community management more fun. This project is for the latter.

More fun than sock puppet accounts, Bot is a tool for kickstarting a new community or bringing an old one together with a shared experience and knowledge.

Bot has customizable triggers that allow it to participate and take actions in your community as a (bot) member.

Bot is best used by experienced Vanilla plugin developers. This guide assumes you understand event handling in Vanilla well.

## Using Bot

You assign a priority order for possible replies. After any event handler sets a reply on Bot, the rest are skipped. By setting the reply to `true` you can prevent _any_ reply being made.

In your `structure()` method, set your replies with their priority level:

`botReply($eventName, $priority);`

Lowest number goes first. Ties may go in any order. If no priority is given, they are prioritized in the order they are set by your plugin. If multiple plugins set unprioritized replies, there may be unpredictable results (plugins may load in any order).

Events are thrown by the `Bot` object. Therefore in your plugin, event handlers for Bot are given the bot instance as `$sender` (the first argument). All the relevant contextual data is available as properties of the bot.

## Replying with Bot

When deciding whether to respond to a particular post, you have some tools available to you via the `Bot` object passed to your event handler:

* `match($text)` returns `true` or `false` whether the body of the post contains the exact `$text`.
* `regex($pattern, $matches)` returns `true` or `false` whether the body of the post contains the regex `$pattern`. Matches are passed back via `$matches`. See `preg_match()`.

When it's time to reply, just use `setReply($string)`. All further reply events are skipped. First reply is best reply.

There's some data & convenience formatting available to you when crafting your reply:

* `mention()` returns a string of an `@` prepended to the username of the author of the triggering post.

Your reply handler should return a fully formatted string of the post you want the bot to reply with. By defaut, Bot expects Markdown. You can change this with `format($newFormat)`.


## Design considerations

* All reply events are fired on every new post until a `true` is returned by one of them. Complex conditions or computations on a busy site could overburden your server.
* Create unique event names so they do not overlap with other plugins.
* Multiple replies may have the same priority. They will be triggered in the order declared, which may be random between plugins.

## Example plugin using Bot

```
<?php
$PluginInfo['shwaipbot'] = array(
   'Name' => 'shwaipbot',
   'Description' => "Example implementation of Bot.",
   'Version' => '1.0',
   'RequiredApplications' => array('Vanilla' => '2.2'),
   'RequiredPlugins' => array('Bot' => '1.0'),
   'MobileFriendly' => TRUE,
   'Author' => "Lincoln Russell",
);

class ShwaipbotPlugin extends Gdn_Plugin {

    /**
     * Simple call and response.
     *
     * User: Shave and a hair cut!
     * Bot: TWO BITS!
     */
    public function bot_shave_handler($bot) {
        if ($bot->match('shave and a hair cut')) {
            $bot->setReply($bot->mention().' TWO BITS!');
        }
    }

    /**
     * Let users send each other beers thru the bot.
     *
     * User: !beer @Lincoln
     * Bot:  /me slides @Lincoln a beer.
     */
    public function bot_sendBeer_handler($bot) {
        if ($bot->pattern('(^|[\s,\.>])\!beer\s@(\w{1,50})\b', $beerTo)) {
            $bot->setReply('/me slides @'.val(2, $beerTo).' a beer.');
        }
    }
    
    /**
     * Just do structure.
     */
    public function setup() {
	     $this->structure();
    }

    /**
     * Register replies.
     */
    public function structure() {
		  botReply('shave');
		  botReply('sendBeer');
    }
}
```