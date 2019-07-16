# SportServ Documentation

### Table Of Contents

- [Overview](#overview)
- [Developing Ads](#developing-ads)
  - [Suggested Workflow](#suggested-workflow)
  - [Development Data](#development-data)
  - [Skeleton Template](#skeleton-template)
  - [Countdown](#countdown)
  - [Beyond Templating](#beyond-templating)
  - [Gotchas](#gotchas)
- [Data Examples](#data-examples)


## Overview

### What is SportServ?
Through our exclusive SportRadar partnership, SportServ syncs in real­time with events for 40+ sports, 800+ leagues, and 400,000+ live events. Those data points allow us to serve specific creative based on any condition (think scoring plays, wins, game times, etc.).

That means personalized ads tailored to the right fans, at the perfect moment in time.

### Understand Your Campaign
It sounds simple, but the first step to a successful campaign is understanding the business needs and goals of the partner. Important things to understand when setting up your campaign:

- All ad creative must be wrapped in HTML; FanServ does not serve image assets on their own. Our images are called from a media bucket on Amazon S3, but can be served from any absolute path.

- SportServ only needs to be called within the HTML if you are pulling in live game data or sport macros into your creative. You can serve basic ad image units around creative conditions like "Scheduled, In Progress, or Completed"  WITHOUT  calling [sportserver] in your HTML. Instead, the SportServ Ad Ops team will use creative conditions while flighting your campaign.

## Developing Ads

### Suggested Workflow
1. Create a flat mock up of your ad in a graphics editing program (photoshop, etc). Consider animations and how live sports data can be used to change the look and message of your ad based on live sports conditions.
2. Take the flat (png, jpg) version of your ad and develop a local static version of the ad using plain html, css, js. At this point the ad won’t be connected to any live data and should follow the same process as developing a static website (albiet with a much smaller target size). You should include “fake” team names and scores in your code. Consider how longer team names will affect the ads’ design.
3. SportServ-ify the ad! At this point, we’re still developing locally but now we want to replace our static content fillers (team names, fake scores, etc) with dynamic data. In order to do that, we’ll want need to include the `SportServer` library that binds data to your ad.
```
<!-- SportServer Library -->
<script src="https://fanserv-static.s3.amazonaws.com/sportserver.js"></script>

<!— Local Development Init script —>
<script>
var sportserver = SportServer.init({
    url: 'http://your-storage-location.com/sample-data/mlb/pre-game.json',
      isGameAd: true,
      renderImmediately: false
});
</script>
```
The `url` property of the `Sportserver.init()` function is a path to the sports data that you’re going to build for. We offer various types of archived sports data at the bottom of this README (if you don’t see what you’re looking for, please reach out to [brad@fanserv.com](mailto:brad@fanserv.com)). At this point in the development cycle, the sports data is still static and won’t update. The purpose of this step is to transform your static html into a Nunjucks template that is bound to the data feed. See the [skeleton template](#skeleton-template) below for an example.
4. The next step is to prepare your ad for the ad server. You will replace the sportserv block below with a `[sportserver2]` macro that will dynamically generate the library and init files based on your campaigns targeting.
```
Before:

<html>
… header elements
<!-- SportServer Library -->
<script src="https://fanserv-static.s3.amazonaws.com/sportserver.js"></script>

<!— Local Development Init script —>
<script>
var sportserver = SportServer.init({
    url: 'http://your-storage-location.com/sample-data/mlb/pre-game.json',
      isGameAd: true,
      renderImmediately: false
});
</script>
… creative code
</html>


After:

<html>
… header elements

[sportserver2]

… creative code
</html>
```
Use the Nunjucks templating language to swap out your dummy data with real live sports data. Don’t worry about matching your ad to a specific team, SportServ will serve your ad to the appropriate team based on your campaign goals.
5. The last step is to take your html and copy and paste it into the adops backend. Please reach out to [brad@fanserv.com](brad@fanserv.com) for access and further details.

### Development Data

In production, SportServ utilizes a server macro `[sportserver]` to dynamically target and link game data to your ad. We simulate that macro while developing ads by adding the following code in the `<head>` of our ad. There are two data sources you can use while developing an ad.

- *Archived Data*: use the [archived json data](https://github.com/fanserv/SportServ-Docs/tree/master/sample-data) in this repo to simulate game conditions. This is the primary way to develop ads because it guarantees game conditions and let's you develop whenever you want.

- *Live Data*: Get up to date [game id's here](https://adops.fanserver.net/sportmanager/public/?page=1). Pass those IDs into the `ad_config` object and recieve LIVE game data while developing your ad. The downside to this technique is that the ad is recieving live data and game conditions can change while you're developing.

###### Archived Data
``` HTML
<!-- SportServ DEV: START -->
  <script src="https://fanserv-static.s3.amazonaws.com/sportserver.js"></script>
  <script>
    var sportserver = SportServer.init({
      url: './sample-data/mlb/pre-game.json',
      isGameAd: true,
      renderImmediately: false
    });
  </script>
<!-- SportServ DEV: END -->
```

###### Live Game Data
``` HTML
<!-- SportServ DEV: START -->
<script>
  var ad_config = {
    server: 'https://sportserver.fanserver.net/sportserver/v1/games',
    game_id: 7798
  }
</script>
<script>var SPORTSERVER_GAME_ID = ad_config.game_id;</script>
<script src="https://s3.amazonaws.com/fanserv-static/production/sportserver.js"></script>
<script>
  var sportserver = SportServer.init({
    url: ad_config.server + ad_config.game_id + '/',
    isGameAd: true,
    renderImmediately: false
  });
</script>
<!-- SportServ DEV: END -->
```


### Skeleton Template
``` HTML
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>FanServ Demo</title>
  <link rel="icon" type="image/x-icon" href="https://fanserv-media.s3.amazonaws.com/client/shared/favicon.ico" />
  <!-- SportServ DEV: START -->
  <script>
    var ad_config = {
      server: 'https://sportserver.fanserver.net/sportserver/v1/games/',
      game_id: 7798
    }
  </script>
  <script>var SPORTSERVER_GAME_ID = ad_config.game_id;</script>
  <script src="https://s3.amazonaws.com/fanserv-static/production/sportserver.js"></script>
  <script>
    var sportserver = SportServer.init({
      url: ad_config.server + ad_config.game_id + '/',
      isGameAd: true,
      renderImmediately: false
    });
  </script>
  <!-- SportServ DEV: END -->
  <script>
  sportserver.events.on(
    SportServer.SPORTSERVER_EVENTS.RENDER, function () {
      // Your custom code here
  });
  </script>
  <script type="text/template">
    <a href="ADD-CLICK-THROUGH-LINK" class="container">
      <!-- PREGAME AD STATE -->
      {% if status == "pregame" %}
      <div class="scoreBox">
        <!-- Away Team -->
        <div class="awayTeam">
          <div class="awayLogo">
            <img class="awayImg" src="https://your-asset-url.com/{{ away.id }}.png">
          </div>
          <div class="awayName">{{ away.alias }}</div>
        </div>
        <!-- Game Info -->
        <div class="gameInfo">
          <div class="outs">{{ countdown("%-H hour%!H %-M minute%!M %-S second%!S") }} until game time!</div>
        </div>
        <!-- Home Team -->
        <div class="homeTeam">
          <div class="homeLogo">
            <img class="homeImg" src="https://your-asset-url.com/{{ home.id }}.png">
          </div>
          <div class="homeName">{{ home.alias }}</div>
        </div>
      </div>
      <!-- LIVE GAME AD STATE -->
      {% elif status == "inprogress" %}
      <div class="scoreBox">
        <!-- Away Team -->
        <div class="awayTeam">
          <div class="awayLogo">
            <img class="awayImg" src="https://your-asset-url.com/{{ away.id }}.png">
          </div>
          <div class="awayName">{{ away.alias }}</div>
          <div class="awayScore">{{ away.points }}</div>
        </div>
        <!-- Game Info -->
        <div class="gameInfo">
          <div class="gameClock">{{ extra.top_or_bottom }}{{ extra.inning }}</div>
          <div class="outs">{{ extra.outs }}</div>
        </div>
        <!-- Home Team -->
        <div class="homeTeam">
          <div class="homeLogo">
            <img class="homeImg" src="https://your-asset-url.com/{{ home.id }}.png">
          </div>
          <div class="homeName">{{ home.alias }}</div>
          <div class="homeScore">{{ home.points }}</div>
        </div>
      </div>
      {% endif %}
    </a>
  </script>
</head>
<body>

<!-- NOTE: SportServ renders the ad into the body -->

</body>
</html>
```

In the ad skeleton above, we're using `{% if status == "pregame" %}` ([if statements](https://mozilla.github.io/nunjucks/templating.html#if)) to target specific game conditions. We then use `{% elif status == "inprogress" %}` to target the live game. The game's _status_ data attribute is the key for most ad transitions.

``` HTML
{% if status == "inprogress" %}
  <div class="scoreBox">
    <div class="primaryTagline">
    {% if home.points > away.points %}
      <!-- Home Team Winning Messaging -->
    {% elif home.points < away.points %}
      <!-- Away Team Winning Messaging -->
    {% elif home.points == away.points %}
      <!-- Tie Game Messaging -->
    {% endif %}
    </div>
    <!-- Away Team -->
    <div class="awayTeam">
      <div class="awayLogo">
        <img class="awayImg" src="https://your-asset-url.com/{{ away.id }}.png">
      </div>
      <div class="awayName">{{ away.alias }}</div>
      <div class="awayScore">{{ away.points }}</div>
    </div>
    <!-- Game Info -->
    <div class="gameInfo">
      <div class="gameClock">{{ extra.top_or_bottom }}{{ extra.inning }}</div>
      <div class="outs">{{ extra.outs }}</div>
    </div>
    <!-- Home Team -->
    <div class="homeTeam">
      <div class="homeLogo">
        <img class="homeImg" src="https://your-asset-url.com/{{ home.id }}.png">
      </div>
      <div class="homeName">{{ home.alias }}</div>
      <div class="homeScore">{{ home.points }}</div>
    </div>
  </div>
{% endif %}
```

In the example above, we introduce dynamic messaging into an "in progress" ad. By using simple [comparison operators](https://mozilla.github.io/nunjucks/templating.html#comparisons) we can transform SportServ ads into real-time dynamic sports ads.


### Countdown

We often want to include a countdown to the start of the game in our ad's pregame state. SportServ includes a handy countdown function based on [jQuery.countdown](http://hilios.github.io/jQuery.countdown/documentation.html). Simply pass the `countdown()` function into nunjucks' bracket (`{{ }}`) to trigger a countdown to the game! Date formatting options are [available here](http://hilios.github.io/jQuery.countdown/documentation.html#formatter).

```
<div class="cd">{{ countdown("%-H hour%!H %-M minute%!M %-S second%!S") }} until game time!</div>
```


### Beyond Templating

Sometimes while building an ad your design will require contacting a 3rd party API, accessing a previously compiled object or simply updating the DOM with game data. To enable this, the `SportServ.init()` method returns an object named `sportserver`. The `sportserver` object includes up to date game data (`sportserver.gamesData`) that you can access directly with javascript and use in your ad.

There are three events that will trigger your code: `RENDER`, `RERENDER` and `GAME_UPDATE`. Those events can be used in the pattern below.

In the following example we reference a value from the `teams` (not shown) object by passing in a dynamic value from the `sportserver.gameData` object. Then we update the DOM direclty with this new data like you would in any other DOM manipulation.

``` javascript
sportserver.events.on(
  SportServer.SPORTSERVER_EVENTS.RENDER, function () {
    var teamInfo = teams[sportserver.gamesData.away.alias];
    var description = "Nike Men's<br/>" + sportserver.gamesData.away.alias +
                      " Home Jersey<br/>" + teamInfo.player +
                      "<br/>#" + teamInfo.number;
    document.getElementById('ad-copy').innerHTML = description;
})
```


### Gotchas
- `HTTPS` is required for asset hosting.

- Certain sports (example: MLB) often start 5-10 min after their scheduled time. Our data feeds update as soon as the game data is available. To account for this we've included a game state called `status: "about-to-start"`. Use this when you don’t want a countdown clock stuck at 00:00:00 while you're waiting for the opening pitch.


## Data Examples
- [MLB - Pregame](https://github.com/fanserv/SportServ-Docs/blob/master/sample-data/mlb/mlb_pre-game.json)
- [MLB - Live Game](https://github.com/fanserv/SportServ-Docs/blob/master/sample-data/mlb/mlb_live-game.json)
- [MLB - Post Game](https://github.com/fanserv/SportServ-Docs/blob/master/sample-data/mlb/mlb_post-game.json)
- [NFL - Pre Game](https://github.com/fanserv/SportServ-Docs/blob/master/sample-data/nfl/nfl_pre-game.json)
- [NFL - Live Game](https://github.com/fanserv/SportServ-Docs/blob/master/sample-data/nfl/nfl_live-game.json)
- [NFL - Post Game](https://github.com/fanserv/SportServ-Docs/blob/master/sample-data/nfl/nfl_post-game.json)
- [NBA - Pre Game](https://github.com/fanserv/SportServ-Docs/blob/master/sample-data/nba/nba_pre-game.json)
- [NBA - Live Game](https://github.com/fanserv/SportServ-Docs/blob/master/sample-data/nba/nba_live-game.json)
- [NBA - Post Game](https://github.com/fanserv/SportServ-Docs/blob/master/sample-data/nba/nba_post-game.json)
- [NHL - Pre Game](https://github.com/fanserv/SportServ-Docs/blob/master/sample-data/nhl/nhl_scheduled.json)
- [NHL - Live Game](https://github.com/fanserv/SportServ-Docs/blob/master/sample-data/nhl/nhl_inprogress.json)
- [NHL - Post Game](https://github.com/fanserv/SportServ-Docs/blob/master/sample-data/nhl/nhl_closed.json)
- [NCAA March Madness - Pre Game](https://github.com/fanserv/SportServ-Docs/blob/master/sample-data/ncaabb/ncaabb_scheduled.json)
- [NCAA March Madness - Live Game](https://github.com/fanserv/SportServ-Docs/blob/master/sample-data/ncaabb/ncaabb_inprogress.json)
- [NCAA March Madness - Post Game](https://github.com/fanserv/SportServ-Docs/blob/master/sample-data/ncaabb/ncaabb_closed.json)
