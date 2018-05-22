# ReactJS项目Unit Test单元测试

1. Import

```javascript
import React from 'react'
import { expect } from 'chai'
import { spy, stub } from 'sinon'
import { Cookies } from 'react-cookie'

import XXXComponent from '../src/components/XXXComponent'
```

2. Setup

```javascript
function setup(propOverrides, renderFunc = mountWithCookie) {
  const context = {
    context: {
      router: {
        // replace: spy(),
      },
    },
    childContextTypes: {
      router: React.PropTypes.object.isRequired,
    }, 
    const preProps = {
        cookies,
        router:{
          setRouteLeaveHook: spy(),
        },
        location: {
          query: {},
        },
        actions: {
          action1: spy(),
          action2: spy(),
          // ...
        },
        config: {
          portalBaseUrl: 'http://localhost:8080',
          countryCode: 'CN',
          // ...
        },
        // ...
   }
   
   const props = Object.assign(preProps, propOverrides)
   const enzymeWrapper = renderFunc(
    <XXXComponent {...props}>
      <XXXComponentUI />
    </XXXComponent>
  , context)

   return {
    props,
    enzymeWrapper,
  }
}
```

   

3. Unit Test Codes

```javascript
describe('<XXXComponent />', () => {
    it('should have a link to add security phone number', () => {
        const preProps = {}
        // spy function
        const dealWithSendValidationCodeStatus = spy(XXXComponentUI.prototype, 'dealWithSendValidationCodeStatus')
    	const { enzymeWrapper } = setup(preProps)
        
        // find component or element
        const XXXComponentUI = enzymeWrapper.find(XXXComponentUI)
        const link = XXXComponentUI.find('.method-button a')
    	expect(link.length).to.equal(1)
        
        // link.simulate('click')
        expect(link.text()).to.contain('Set')
        
        // set props
        enzymeWrapper.setProps({
          msisdn: '138*****000',
        })
        
        // set state
        enzymeWrapper.setState({
          step: 1,
        })
        
        // operate input
        const input = enzymeWrapper.find('.dialog-input input')
        expect(input.exists()).to.equal(true)
            input.simulate('change', {
              target: {
                value: '1234',
              },
            })
        })
    
    	// value of state
    	expect(enzymeWrapper.state().validationCode).to.contains('1234')
    	
    	// also we can spy functions of window
    	window.clearTimeout = spy()
    	// do something
    	expect(window.clearTimeout.called).to.equal(true)
}
```

4. Configuration gulp tasks

```javascript
// gulp files
function karmaFinishHandler(done) {
  return failCount => {
    done(failCount ? new Error(`Failed ${failCount} tests.`) : null)
  }
}

function karmaSingleRun(done) {
  const configFile = path.join(process.cwd(), 'build', 'karma.conf.js')
  const karmaServer = new karma.Server({
    configFile,
    singleRun: false,
  }, karmaFinishHandler(done))
  karmaServer.start()
}

gulp.task('karma:single-run', karmaSingleRun)
gulp.task('test', ['karma:single-run'])
```

5. Configuration for karma

```javascript
module.exports = function (config) {
  config.set({

    // base path that will be used to resolve all patterns (eg. files, exclude)
    basePath: './',

    // frameworks to use
    // available frameworks: https://npmjs.org/browse/keyword/karma-adapter
    frameworks: ['mocha'],

    // list of files / patterns to load in the browser
    files: [
      '../test/ut/index.js',
    ],


    // list of files to exclude
    exclude: [
      '../src/**/*.scss',
    ],

    // conifguration for webpack
    webpack: webpackConfig(),

    // preprocess matching files before serving them to the browser
    // available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
    preprocessors: {
      '../test/ut/index.js': ['webpack'],
    },

    browserNoActivityTimeout: 20000,
    // test results reporter to use
    // possible values: 'dots', 'progress'
    // available reporters: https://npmjs.org/browse/keyword/karma-reporter
    reporters: ['mocha', 'coverage-istanbul'],

    // web server port
    port: 9876,

    // enable / disable colors in the output (reporters and logs)
    colors: true,

    // level of logging
    // possible values: config.LOG_DISABLE || config.LOG_ERROR || config.LOG_WARN || config.LOG_INFO || config.LOG_DEBUG
    logLevel: config.LOG_INFO,

    // enable / disable watching file and executing tests whenever any file changes
    autoWatch: true,

    // start these browsers
    // available browser launchers: https://npmjs.org/browse/keyword/karma-launcher
    browsers: ['Chrome'],

    // Continuous Integration mode
    // if true, Karma captures browsers, runs the tests and exits
    singleRun: false,

    // Concurrency level
    // how many browser should be started simultaneous
    concurrency: Infinity,

    coverageIstanbulReporter: {
      reports: ['text-summary', 'html'],
      dir: path.join(__dirname, '../coverage'),
      fixWebpackSourcePaths: true,
    },
    webpackServer: {
      noInfo: true, //please don't spam the console when running in karma!
    },
    plugins: [
      'karma-mocha',
      'karma-mocha-reporter',
      'karma-coverage-istanbul-reporter',
      'karma-webpack',
      'karma-sourcemap-loader',
      'karma-chrome-launcher',
    ],
  })
}
```

6. Webpack test cofiguration

```javascript
const autowatch = process.env.npm_config_argv.indexOf('--watch') !== -1
module.exports = function() {
  let config = {
    devtool: 'inline-source-map',
    resolve: {
      extensions: ['.jsx', '.js', '.json', '.scss'],
    },
    module: {
      loaders: [
        {
          test: /.jsx?$/,
          loader: 'babel-loader',
          exclude: /node_modules/,
          query: {
            presets: ['es2015', 'stage-0', 'react'],
          },
        },
        {
          test: /(\.css|\.scss)$/,
          loaders: 'css-loader!postcss-loader!sass-loader',
        },
        {
          test: /\.svg(\?v=\d+\.\d+\.\d+)?$/,
          loader: 'url-loader?limit=10000&minetype=image/svg+xml',
        },
        {
          test: /\.ttf(\?v=\d+\.\d+\.\d+)?$/,
          loader: 'url-loader?limit=10000&minetype=application/octet-stream',
        },
        { test: /\.(jpe?g|png|gif)$/i, loader: 'file-loader?name=[name].[ext]' },
      ],
    },
    externals: {
      'react/addons': true,
      'react/lib/ExecutionEnvironment': true,
      'react/lib/ReactContext': true,
      'react-test-renderer/shallow': true,
      'react-dom/test-utils': true,
    },
    plugins: [
      new webpack.LoaderOptionsPlugin({
        options: {
          postcss: () => [autoprefixer],
        },
      }),
      new webpack.DefinePlugin({
        'process.env._LOGGING_LEVEL': '1',
        'process.env.NODE_ENV': '"test"',
      }),
    ],
  }
  if (!autowatch) {
    config.module.loaders.unshift({
      test: /\.jsx?$/,
      include: path.resolve('src/'),
      exclude: [path.join(process.cwd(), 'src/locales'), path.join(process.cwd(), 'src/utilities')],
      loader: 'istanbul-instrumenter-loader',
      options: {
        esModules: true,
      },
    })
  }
  return config
}   
```

7. Run Test

```shel
npm run test
```

   