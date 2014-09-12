## Numeral-js fork

```
npm install gammanumeral --save
```

## What I have done

1- **Added language setting to Numeral instance**: This way I can have many instances, each one in a specific language. Ability to change instance's language in runtime.

2- **Created NumeralFactory**: A numeral factory holds default options so any numeral created from the same factory is created alike. Example: 

```javascript
var numeral = require("numeral");

var ptBrFactory = new numeral.newFactory({language: 'pt-br'});
var oneThousand = ptBrFactory.newNumeral(1000);

console.log(oneThounsand.language()); //pt-br
```

3- **Created numeral.expressBind**: A very small helper for express for those like me who want to use Numeral for i18n purposes. Code is very simple:

```javascript
numeral.expressBind = function(app){
     if (!app) {
         return;
     }
         
     app.use(function(req, res, next) {
         req.numeralFactory = new NumeralFactory();
         req.numeral = req.numeralFactory.newNumeral; 
             
         next();
     });
};
```

4- **Created a way to load available languages that weren't exposed (one line change)**: 
```javascript
if(!languages[key]) {
    //throw new Error('Unknown language : ' + key); this line was replaced by the one above
    loadLanguage(key, require(path.join(__dirname, '/languages/' + key + '.js'))); //still throws if language key is not found
}
``` 

5- **Created a way to expose available languages and loadedLanguages**: This can be achieved both from `numeral` top level object or any factory. See line 472 to 488. 

Please, leave your comments on my changes. I would like to reinforce that this is my first change to someone else's codebase so I am really new to contributing to open source projects. Positive criticism is welcome!

### Usage Example (simply copy/paste the code above)
Remember to create a new folder then run `npm install express gammanumeral`

```javascript
var express = require("express"),
    app = express(),
    gammanumeral = require("gammanumeral");

gammanumeral.expressBind(app);

app.use(express.cookieParser());
app.use(express.session({
    secret: "thisIsSecret"
}));

app.get("/", function(req, res, next){
    if(req.session.language) req.numeralFactory.language(req.session.language);
    
    var html = "";
    html += "numeral(1000.234).format('$0,0.00') === " + req.numeral(1000.234).format("$0,0.00") + "<br />";
    html += "numeral(10000).format('0,000.00') === " + req.numeral(10000).format("0,000.00") + "<br />";
    html += "numeral(0.974878234).format('0.000%') === " + req.numeral(0.974878234).format("0.000%");
    
    res.send(html);
});

app.get("/i18n/:language", function(req, res, next){
    req.session.language = req.params.language;
    res.redirect("/");
});

app.get("/i18n", function(req, res, next){
   req.numeralFactory.availableLanguages(function(err, languages){
      if(err) return next(err);
      
      var html = "";
      languages.forEach(function(language, index){
          html += (index + 1).toString() + ".&nbsp;<a href='/i18n/" + language + "'>" + language + "</a><br />";
      });
      
      res.send(html);
   }); 
});

app.listen(3000);
```
