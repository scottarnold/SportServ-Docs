# SportServ Documentation

## Table Of Contents

- SportServ SDK
- Anatomy of Ad (sportserv feed + templating)
- Handling Assets
- Data Examples:
	- [Pregame](#pregame)
	- [Live Game](#example-live_game)
	- [Post Game](#example-post_game)

## Summary
Creating SportServ ads is as easy as creating a minitiure website!Q Our ads are standard HTML5 ads and give you complete control over the HTML, CSS and JS that your ad uses.


## Suggested Workflow
1. Create flat mock ups of your creative. Consider animations and how live sports data can be used to change the look and message of your ad based on live sports conditions. 
2. Turn that mock up into html, css and js. Make up sample dummy data to fill in the live aspects of your 
3. SportServ-ify the ad! Use the [Nunjucks](’https://mozilla.github.io/nunjucks/templating.html')  templating language to swap out your dummy data with real live sports data. Don’t worry about matching your ad to a specific team, SportServ will serve your ad to the appropriate team based on your campaign goals.


#### SportServ Library

###### Production:

``` HTML
[sportserver]
```

In production this macro is expanded to the following. Production enables dynamic game targetting...

###### Development:

``` HTML
<script src="https://s3.amazonaws.com/fanserv-static/dev/nunjucks.min.js"></script>
<script src="https://s3.amazonaws.com/fanserv-static/dev/sportserver.js"></script>
<script>
  SportServer.init({
    url: "http://adserver-dev-env.elasticbeanstalk.com/sportserver/v1/games/5788/", isGameAd: true 
  })
</script>
```

## SportServ Initializer Options

-  `isGameAd` (boolean) - if the data received is for a game. It will have flattened data if so.
- `url` (string) - URL to request data from 
- `gameId` (int) - id of a game, will be used to generate a url
- `teamId` (int) - id of a team, will be used to generate a url
- `filters` (func) - filters to add to the templating engine

```javascript
<script>
	var sportserver = SportServer.init({
     url: 'https://adserver.fanserver.net/sportserver/v1/games/' + SPORTSERVER_GAME_ID + '/',
	  isGameAd: true,
	  renderImmediately: true
	});
</script>
```


## SportServ Load States

- `SPORTSERVER_EVENTS.RENDER` - An update occurred in the game associated with the ad
- `SPORTSERVER_EVENTS.GAME_UPDATE` - The ad was rendered for the first time
- `SPORTSERVER_EVENTS.RERENDER` - The ad was rerendered



#####  Render

```html
sportserver.events.on(
  SportServer.SPORTSERVER_EVENTS.RENDER, function () {
    console.log('initial render');
})
```

##### Game Update
```html
sportserver.events.on(
  SportServer.SPORTSERVER_EVENTS.GAME_UPDATE, function () {
    console.log('game update');
    Animation.start();
})
```

##### RERENDER

```html
sportserver.events.on(
  SportServer.SPORTSERVER_EVENTS.RERENDER, function () {
    console.log('re-rendered');
})
```



## Game States (MLB) - Discontinued in V2

|Game State   |Syntax                                  |Descreiption        |
|-------------|----------------------------------------|--------------------|
|No Game      |`{% block nogame %}...{% endblock %}`   |Shows when no scheduled game  |
|Pregame      |`{% block pregame %}...{% endblock %}`  |Shows x hours before the game|
|Live Game    |`{% block ingame %}...{% endblock %}`   |includes live score |
|Game Over    |`{% block pastgame %}...{% endblock %}` |game over           |

## Live game state logic examples

We combine simple template logic with our game data to further drill down into specific game states 

##### Tie Game
``` javascript
{% if home.points == away.points %}
```

##### Home Team Down
``` javascript
{% if home.points < away.points %}
```

##### Home Team Up
``` javascript
{% if home.points > away.points %}
```

##### Home Team Up more than 5
``` javascript
{% if (home.points - away.points) > 5  %}
```


## Data Structures

### MLB

###### PREGAME

``` JSON
{
  "away": {
    "alias": "SD",
    "name": "Padres",
    "colors": [ "#05143f" ],
    "logo": "https://sportserver-media.s3.amazonaws.com/media/teams_logos/San_Diego_Padres.png",
    "id": 1160
  },
  "is_full_coverage": true,
  "status": "scheduled",
  "scheduled": "2016-07-22T23:05:00Z",
  "league": "MLB",
  "id": 8394,
  "hash": "04b7b2250b8927e1657a41f1c3290fce",
  "home": {
    "alias": "WSH",
    "name": "Nationals",
    "colors": [
      "#cc1242"
    ],
    "logo": "https://sportserver-media.s3.amazonaws.com/media/teams_logos/Washington_Nationals.png",
    "id": 1169
  }
}
```

* Note: MLB games often start 5-10 min after their scheduled time. We’ve included the state `status: "about-to-start"` for these situations. You can use this when you don’t want a countdown clock stuck at 00:00:00 before the opening pitch.


###### LIVE GAME

``` JSON
{
  "status": "inprogress",
  "home": {
    "stats": {
      "hits": 0,
      "score_by_inning": {
        "2": "-",
        "5": "-",
        "9": "-",
        "1": "X",
        "4": "-",
        "7": "-",
        "8": "-",
        "6": "-",
        "3": "-"
      },
      "errors": 0
    },
    "players": [],
    "colors": [
      "#fdb827"
    ],
    "points": 0,
    "logo": "https://sportserver-media.s3.amazonaws.com/media/teams_logos/Pittsburgh_Pirates.png",
    "alias": "PIT",
    "name": "Pirates",
    "id": 1145
  },
  "scheduled": "2016-07-22T23:05:00Z",
  "away": {
    "stats": {
      "hits": 0,
      "score_by_inning": {
        "2": "-",
        "4": "-",
        "6": "-",
        "1": "0",
        "5": "-",
        "7": "-",
        "8": "-",
        "3": "-",
        "9": "-"
      },
      "errors": 0
    },
    "players": [],
    "colors": [
      "#cc1242"
    ],
    "points": 0,
    "logo": "https://sportserver-media.s3.amazonaws.com/media/teams_logos/Philadelphia_Phillies.png",
    "alias": "PHI",
    "name": "Phillies",
    "id": 1148
  },
  "hash": "0f5246013777a282e5e7e52042ff1caf",
  "events": [],
  "extra": {
    "top_or_bottom": "top",
    "count": {
      "outs": 0,
      "pitch_count": 2,
      "balls": 0,
      "strikes": 2
    },
    "inning": "1st",
    "period": 1
  },
  "is_full_coverage": true,
  "league": "MLB",
  "id": 8200,
  "period": 1
}
```
