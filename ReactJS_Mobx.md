# ReactJS + Mobx项目简单搭建

1. Server & Mock

```Javascript
'use strict';
const path = require('path')
const WebpackDevServer = require('webpack-dev-server')
const webpack = require('webpack')
// const dev_config = require('../webpack-dev.config')
// const prod_config = require('../webpack.config')
const config = require('../webpack-dev.config')
const express = require('express')
const proxy = require('proxy-middleware');
const renderIndex = require('../lib/render-index');
const url = require('url');
const bodyParser = require('body-parser');
const middleware = require('webpack-dev-middleware');
const hotMiddleware = require('webpack-hot-middleware')
const compress = require('compression')
const indexMarkup = renderIndex({dev: true});
const port = process.env.PORT || 12780;
const app = express()
app.use(compress())

const domainHost = 'http://iam.westus.cloudapp.azure.com'
app.use(
  '/remote',
  proxy(url.parse(${domainHost}:27020))
)

const imgOptions = {
  etag: false,
  index: false,
  redirect: false,
  setHeaders: function (res) {
    // res.set('content-type', 'image/jpeg');
  }
}
app.use('/static/img', express.static(path.join(__dirname, '../assets/img'), imgOptions))

const options = {
  dotfiles: 'ignore',
  etag: false,
  extensions: ['json'],
  index: false,
  redirect: false,
  setHeaders: function (res) {
    res.set('content-type', 'application/json');
  }
}

app.use('/api', express.static(path.join(__dirname, 'mock'), options))
app.use(bodyParser.json()); // for parsing application/json
app.use(bodyParser.urlencoded({ extended: true })); // for parsing application/x-www-form-urlencoded
// app.use('/', express.static(path.join(__dirname)))
const compiler = webpack(config)

app.use(middleware(compiler, {
  publicPath: config.output.publicPath,
  contentBase : path.resolve(__dirname, 'app'),
  hot: true,
  quiet: true,
  lazy: false,
  stats: {colors: true},
  noInfo: true,
}))

app.use(hotMiddleware(compiler, {
  path: '/__webpack_hmr',
}))

app.use( express.static( path.resolve(__dirname, 'public') ) )

app.get('*', function(req, res) {
  res.send(indexMarkup);
});

app.listen(port, function(err) {
  if (err) { return console.log(err); }
  console.log('Listening at http://0.0.0.0:%d', port);
});

var server = new WebpackDevServer(webpack(config), {
  publicPath: config.output.publicPath,
  hot: true,
  stats: {colors: true}
});

server.listen(3002, function(err) {
  if (err) { return console.log(err); }
  console.log('Server started')
  // app.listen(port, function(err) {
  //   if (err) { return console.log(err); }
  //   console.log('Listening at http://0.0.0.0:%d', port);
  // });
});
```



2. Privider

   ```javascript
   import { Router } from 'react-router'
   
   class Root extends Component {
     render() {
       return (
         <div>
           <Provider {...store}>
             <Router routes={routes} history={browserHistory} />
           </Provider>
         </div>
       )
     }
   }
   ```

   

3. Router

```javascript
import React, { Component } from 'react';
import { Route, IndexRoute } from 'react-router'
// ...
export default (
  <Route>
    <Route path="/" component={App}>
      <IndexRoute component={MainMenu} />
      <Route path="/menu" component={MainMenu} />
      <Route path="/callback" component={Callback} />
    </Route>
    <Route path="/main" component={MainFrame}>
    </Route>
    <Route path="*" component={NotFound} />
  </Route>
)
```



4. Index

```javascript
'use strict';

var fs = require('fs');
var path = require('path');

var indexPath = path.resolve(__dirname, '..', 'index.html');
var indexMarkup = fs.readFileSync(indexPath).toString();

function renderIndex(options) {
  if (!options) { options = {}; }

  var appMarkup = options.dev ?
    '' :
    options.appMarkup || '';

  return indexMarkup
    .replace('<!-- app -->', appMarkup)
    .replace(
      '<!-- script -->',
      options.dev ?
        '<script src="/public/bundle.js"></script>' :
        '<script src="/bundle.js"></script>')
    .replace(
      '<!-- style -->',
      options.dev ?
        '' :
        '<link rel="stylesheet" href="/public/style.css" />');
}

module.exports = renderIndex;
```



```javascript
render() {
    return (
      <div className="app">
        {this.props.children}
      </div>
    )
  }
```



5. Modal & Store

```javascript
class Authorize {
  store

  @observable code

  constructor(store, { code }) {
    this.store = store
    this.code = code
  }

  @action setValue(columnName, val) {
    this[columnName] = val
  }

  toJS() {
    return {
      code: this.code,
    }
  }

  static fromJS(store, { code }) {
    return new Authorize(store, { code })
  }
}

export default Authorize
```



```javascript
class AuthorizeStore {
  @observable authorize

  @action initAuthorize(){
    this.authorize = new Authorize(this, {
      code: null,
    })
  }

  toJS() {
    return this.authorize.toJS()
  }

  static fromJS({code}) {
    const authorizeStore = new AuthorizeStore()
    authorizeStore.authorize = Authorize.fromJS(authorizeStore, {
      code: code || null,
    })
    return authorizeStore
  }
}

export default AuthorizeStore
```

6. observer & injection

```javascript
@inject('patientStore')
@observer
class PatientTableContainer extends Component {
	// ...
}
```

7. Configuration & Run

```json
"start": "cross-env NODE_ENV=development node bin/dev-server",
"production": "cross-env NODE_ENV=production node bin/pro-server",
"build": "cross-env NODE_ENV=production webpack -p",
"server": "cross-env NODE_ENV=production npm run build && npm run production "
```

```shell
npm start
# or
npm run production
# or build
npm run build
```



p.s.

下次我们研究如何进行Unit Test。