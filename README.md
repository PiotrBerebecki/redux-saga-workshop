# Redux Saga Workshop

# Step 1

Goal: get webpack and babel running

  - Set up a new folder to work in
  - `npm init` and press ENTER all the way
  - `npm install --save-dev babel-core babel-loader babel-preset-es2015 eslint eslint-config-mailonline rimraf webpack webpack-dev-server`
    - `webpack`: to bundle our javascript
    - `webpack-dev-server`: to run a server on localhost which bundles and automatically refreshes
    - `babel-core` and `babel-loader`: so that webpack can load js files through babel
    - `babel-preset-es2015`: a rather broad babel preset, just to start with, will transpile a lot of stuff
    - `eslint` and `eslint-config-mailonline`: to have some linting from the get-go
    - `rimraf`: to remove the `dist` folder before building
  - Create a `.gitignore` file with these contents:
    ```
    *.log
    *.swo
    *.swp
    *.lock
    .DS_Store
    .idea/
    .scripts/
    .vagrant/
    .nyc_output/
    .vscode/
    dist/
    coverage/
    dev/
    node_modules/
    ```
  - Create `.eslintrc.json` with these contents:
    ```json
    {
      "extends": "mailonline",
      "root": true
    }
    ```
  - Create `.babelrc` with these contents:
    ```json
    {
      "presets": ["es2015"]
    }
    ```
  - Create `src/index.js` with the following contents:
    ```js
    /* eslint-disable no-console */
    console.log('Hello world!');
    ```
  - Create `webpack.config.babel.js` with these contents:
    ```js
    import path from 'path';

    export default {
      devServer: {
        contentBase: [
          'demo/'
        ],
        host: '0.0.0.0',
        inline: true,
        publicPath: '/dist/',
        watchContentBase: true
      },
      entry: {
        index: './src/index.js'
      },
      module: {
        rules: [
          {
            loader: 'babel-loader',
            test: /\.js$/
          }
        ]
      },
      output: {
        filename: '[name].js',
        path: path.join(__dirname, 'dist')
      }
    };
    ```
  - Create a `demo` folder with an `index.html` file:
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
      <title>Dev server</title>
    </head>
    <body>
      <script src="/dist/index.js" defer></script>
    </body>
    </html>
    ```
  - Modify your `package.json` `scripts` section:
    ```json
    "scripts": {
      "prebuild": "rimraf ./dist",
      "build": "webpack -p",
      "start": "webpack-dev-server",
      "lint": "eslint --ignore-path .gitignore '**/*.js'"
    }
    ```
  - Run `npm run start` and open `localhost:8080` (or whichever port webpack-dev-server gives you) in your browser, and you should see `Hello world!` in your console. Changing the JS or the html will auto-refresh your browser tab
  - Run `npm run build` and you'll see your bundled script in `dist/index.js`

Now you're ready for [Step 2](../step2)

# Step 2

Goal: add react and css modules

  - `npm install --save-dev autoprefixer babel-preset-react css-loader node-sass postcss-loader prop-types react react-dom sass-loader style-loader stylelint stylelint-config-standard`
    - `react` and `react-dom`: to use react
    - `sass-loader` and `node-sass`: for webpack to load SCSS files and convert them into CSS
    - `css-loader`: for webpack to load CSS files and, in our case, to parse them into CSS modules
    - `style-loader`: for webpack to bundle CSS into our javascript, and inline it into the page on demand (to extract all CSS into an external CSS file, you'd need to use ExtractTextPlugin)
    - `stylelint` and `stylelint-config-mailonline`: to lint our CSS
    - `babel-preset-react`: to add some plugins to our babel config, especially to transpile JSX syntax
    - `autoprefixer` and `postcss-loader`: to post-process CSS, adding vendor prefixes to our rules
  - Add `babel-preset-react` to our babel config in `.babelrc`:
    ```json
    {
      "presets": ["es2015", "react"]
    }
    ```
  - Create `.stylelintrc.json`:
    ```json
    {
      "extends": "stylelint-config-standard"
    }
    ```
  - Modify `.eslintrc.json` to add `mailonline/react`:
    ```json
    {
      "extends": ["mailonline", "mailonline/react"],
      "root": true
    }
    ```
  - Create `postcss.config.js` (don't worry too much about it right now):
    ```js
    /* eslint-disable import/no-commonjs, import/unambiguous, filenames/match-regex */
    const autoprefixer = require('autoprefixer');

    module.exports = {
      plugins: [
        autoprefixer({
          browsers: [
            '> 1%',
            'last 3 versions',
            'iOS > 8',
            'not ie < 10'
          ]
        })
      ]
    };
    ```
  - Modify your `module -> rules` array in `webpack.config.babel.js` to add a loader for `.scss` files:
    ```js
    ...
    module: {
      rules: [
        {
          loader: 'babel-loader',
          test: /\.js$/
        },
        {
          test: /\.scss$/,
          use: [
            'style-loader?sourceMap',
            {
              loader: 'css-loader',
              options: {
                importLoaders: 3,
                localIdentName: '[local]__[hash:base64:5]',
                modules: true,
                sourceMap: true
              }
            },
            'postcss-loader?sourceMap',
            'sass-loader?sourceMap'
          ]
        }
      ]
    },
    ...
    ```
  - Replace your `lint` script in `package.json` with this:
    ```json
    "lint-scripts": "eslint --ignore-path .gitignore '**/*.js'",
    "lint-styles": "stylelint --ignore-path .gitignore '**/*.{scss,sass,css}'",
    "lint": "npm run lint-scripts && npm run lint-styles && npm run lint-json",
    ```
  - Create `src/styles.scss`:
    ```css
    .container {
      padding: 16px;
      background: #eee;
      border: 1px solid black;
    }
    ```
  - Modify `src/index.js`:
    ```js
    import React from 'react';
    import {render} from 'react-dom';
    import styles from './styles.scss';

    render(
      <div className={styles.container}>Hello world!</div>,
      document.querySelector('[data-react-workshop]')
    );
    ```
  - Modify `demo/index.html`, add this before the `script` element, in the body:
    ```html
    <div data-react-workshop></div>
    ```
  - Run `npm run start` and open `localhost:8080` (or whichever port applies), you should see a grey box with a black border and "Hello world!" inside


  # Step 3

  Goal: build our actual app with redux and redux-saga

    - `npm install --save-dev babel-plugin-transform-async-to-generator babel-plugin-transform-runtime babel-preset-stage-3 react-redux redux redux-saga whatwg-fetch`
      - `babel-plugin-transform-async-to-generator`: to use async/await
      - `babel-plugin-transform-runtime`: in a real production environment we'd only use this for tests, and babel-preset-env + useBuiltIns + babel-polyfill for the actual bundle, but for simplicity's sake we'll just use this one here on its own - please do look into babel-preset-env
      - `babel-preset-stage-3`: for object rest spread (`{...}`)
      - `react-redux`: to connect redux state to react components
      - `redux-saga`: to handle side effects in our app (e.g. fetching from the github API)
    - Add `stage-3` preset and `transform-runtime` plugin to our `.babelrc` config:
      ```json
      {
        "presets": ["es2015", "stage-3", "react"],
        "plugins": [
          ["transform-runtime", {
            "helpers": false,
            "polyfill": false,
            "regenerator": true
          }]
        ]
      }
      ```
    - Start with the action creators - create `src/actions/repos.js`:
      ```js
      export const types = {
        ERROR_REQUESTING_REPOS: 'ERROR_REQUESTING_REPOS',
        RECEIVED_REPOS: 'RECEIVED_REPOS',
        REQUEST_REPOS: 'REQUEST_REPOS'
      };

      export const errorRequestingRepos = (error) => ({
        payload: {
          error
        },
        type: types.ERROR_REQUESTING_REPOS
      });

      export const receivedRepos = (repos) => ({
        payload: {
          repos
        },
        type: types.RECEIVED_REPOS
      });

      export const requestRepos = (organization) => ({
        payload: {
          organization
        },
        type: types.REQUEST_REPOS
      });
      ```
    - Now the reducer - create `src/reducers/repos.js`:
      ```js
      import {types} from '../actions/repos';

      const {
        ERROR_REQUESTING_REPOS,
        RECEIVED_REPOS,
        REQUEST_REPOS
      } = types;

      export const initialState = {
        error: null,
        fetching: false,
        repos: null
      };

      export default function repos (state = initialState, action) {
        switch (action.type) {
        case ERROR_REQUESTING_REPOS:
          return {
            ...state,
            error: action.payload.error
          };
        case RECEIVED_REPOS:
          return {
            ...state,
            fetching: false,
            repos: action.payload.repos
          };
        case REQUEST_REPOS:
          return {
            ...state,
            fetching: true,
            repos: null
          };
        default:
          return state;
        }
      }
      ```
    - And a root reducer which won't make much sense here, but would combine multiple ones in a real app - create `src/reducers/index.js`:
      ```js
      import {combineReducers} from 'redux';
      import repos from './repos';

      const reducers = combineReducers({
        repos
      });

      export default reducers;
      ```
    - Add a `fetch` polyfill to our app - modify `src/index.js` and add this to the very top:
      ```js
      // eslint-disable-next-line import/no-unassigned-import
      import 'whatwg-fetch';
      ```
    - Let's create an API for fetching the repos - create `src/api/repos.js`:
      ```js
      export const fetchByOrg = async (organization) => {
        const endpoint = `https://api.github.com/orgs/${organization}/repos`;
        const response = await fetch(endpoint);
        const json = await response.json();

        if (response.status < 200 || response.status >= 400) {
          const error = new Error(json && json.message || response.statusText);

          error.response = response;
          throw error;
        }

        return json.map(
          // eslint-disable-next-line id-match
          ({name, html_url}) => ({
            name,
            url: html_url
          })
        );
      };
      ```
    - Now it's time to create our saga - create `src/sagas/repos.js`:
      ```js
      import {call, fork, put, takeLatest} from 'redux-saga/effects';
      import {fetchByOrg} from '../api/repos';
      import {
        errorRequestingRepos,
        receivedRepos,
        types
      } from '../actions/repos';

      export const fetchReposFromApi = function *(action) {
        try {
          const {payload: {organization}} = action;

          const reposFromApi = yield call(fetchByOrg, organization);

          yield put(receivedRepos(reposFromApi));
        } catch (error) {
          yield put(errorRequestingRepos(error));
        }
      };

      const watchRequestRepos = function *() {
        yield takeLatest(types.REQUEST_REPOS, fetchReposFromApi);
      };

      export default function *repos () {
        yield [
          fork(watchRequestRepos)
        ];
      }
      ```
    - Next, we create a root saga, much like the root reducer - create `src/sagas/index.js`:
      ```js
      import {fork} from 'redux-saga/effects';
      import repos from './repos';

      export default function *sagas () {
        yield [
          fork(repos)
        ];
      }
      ```
    - The only thing left for our redux side of the app is, first, creating a store - create `src/store/configureStore.js`:
      ```js
      import {applyMiddleware, createStore, compose} from 'redux';
      import createSagaMiddleware from 'redux-saga';
      import rootReducer from '../reducers';
      import rootSaga from '../sagas';

      export default function configureStore (preloadedState) {
        const sagaMiddleware = createSagaMiddleware();
        const store = createStore(
          rootReducer,
          preloadedState,
          compose(
            applyMiddleware(sagaMiddleware),
            window && window.devToolsExtension ?
              window.devToolsExtension() :
              (passThrough) => passThrough
          )
        );

        sagaMiddleware.run(rootSaga);

        return store;
      }
      ```
    - And, finally, wiring it into our app via the `react-redux` `Provider` HOC - modify `src/index.js`:
      ```js
      // eslint-disable-next-line import/no-unassigned-import
      import 'whatwg-fetch';
      import React from 'react';
      import {render} from 'react-dom';
      import {Provider} from 'react-redux';
      import configureStore from './store/configureStore';
      import styles from './styles.scss';

      const store = configureStore();

      render(
        <Provider store={store}>
          <div className={styles.container}>Hello world!</div>
        </Provider>,
        document.querySelector('[data-react-workshop]')
      );
      ```

  We now have a perfectly workable Redux state tree, with sagas running... except we have no UI components to:
    1) Dispatch the action that will trigger the saga that fetches the repos, and
    2) Display the information conveyed by the state tree to the user

  So let's create some components to do just that.

    - Create `src/components/GetReposButton/index.js`:
      ```js
      import React from 'react';
      import PropTypes from 'prop-types';
      import {connect} from 'react-redux';
      import {requestRepos} from '../../actions/repos';
      import styles from './styles.scss';

      const HARDCODED_ORG_NAME = 'github';

      const GetReposButton = ({dispatch, fetching}) => {
        const handleOnClick = () => dispatch(requestRepos(HARDCODED_ORG_NAME));

        return (
          <button
            className={styles.button}
            disabled={fetching}
            onClick={!fetching && handleOnClick}
          >
            Get Repos
          </button>
        );
      };

      GetReposButton.propTypes = {
        dispatch: PropTypes.func.isRequired,
        fetching: PropTypes.bool.isRequired
      };

      const mapStateToProps = (state) => ({
        fetching: state.repos.fetching
      });

      export {GetReposButton as GetReposButtonPureComponent};
      export default connect(mapStateToProps)(GetReposButton);
      ```
    - And its corresponding `src/components/GetReposButton/styles.scss`:
      ```css
      .button {
        background: #4cf;
        font-weight: bold;
        border: 0;
        appearance: none;
        padding: 16px;

        &[disabled] {
          background: #999;
          color: white;
        }
      }
      ```
    - Now to display some results - create `src/components/RepoList/index.js`:
      ```js
      import React from 'react';
      import PropTypes from 'prop-types';
      import {connect} from 'react-redux';

      const RepoList = ({fetching, repos}) => {
        if (fetching) {
          return <p>Loading...</p>;
        }

        if (!repos) {
          return <p>No results</p>;
        }

        return (
          <ul>
            {
              repos.map(
                ({name, url}) =>
                  <li key={name}>
                    <a href={url}>{name}</a>
                  </li>
              )
            }
          </ul>
        );
      };

      RepoList.propTypes = {
        fetching: PropTypes.bool.isRequired,
        repos: PropTypes.array
      };

      RepoList.defaultProps = {
        repos: null
      };

      const mapStateToProps = (state) => ({
        fetching: state.repos.fetching,
        repos: state.repos.repos
      });

      export {RepoList as RepoListPureComponent};
      export default connect(mapStateToProps)(RepoList);
      ```
    - And now we just wire them together into the root app component - modify `src/index.js`:
      ```js
      // eslint-disable-next-line import/no-unassigned-import
      import 'whatwg-fetch';
      import React from 'react';
      import {render} from 'react-dom';
      import {Provider} from 'react-redux';
      import GetReposButton from './components/GetReposButton';
      import RepoList from './components/RepoList';
      import configureStore from './store/configureStore';
      import styles from './styles.scss';

      const store = configureStore();

      render(
        <Provider store={store}>
          <div className={styles.container}>
            <GetReposButton />
            <RepoList />
          </div>
        </Provider>,
        document.querySelector('[data-react-workshop]')
      );
      ```
    - Run `npm run start` and open `localhost:8080` (or whichever port) - click on the button
    - Congratulations! You should have a fully running react/redux/redux-saga application there, unless something went horribly wrong.

  Keep in mind that there's millions of equally valid ways to create a react/redux application - the flexibility offered by these libraries is immense. You'll have to experiment and pick and choose to find what works for you.

  Now, no application is complete without tests - so let's see to that in [Step 4](../step4)

  # Step 4

  Goal: add tests with jest

  In these kinds of apps, it's a good idea to make components as dumb as possible - they take in state, and dispatch actions. Some inevitable exceptions will arise, but the general rule should be to keep them as simple as possible. The real meat-and-bones will be in the sagas, actions and reducers - so our focus while testing should be on those.

  I'll just provide example tests for the main saga, the reducer, and the components (using both enzyme and jest snapshots to showcase both approaches, which are not exclusive at all).

    - `npm install --save-dev babel-jest jest identity-obj-proxy enzyme react-test-renderer`
      - `babel-jest`: for jest to run js files through babel
      - `jest`: to run our test suite
      - `identity-obj-proxy`: we're not going through webpack, so we need something to replace CSS modules - this is just a proxy that, when trying to access any property, will return the key as a string
    - Modify your `package.json`, adding a new `jest` section, and a `test` script in `scripts`:
      ```json
      {
        ...
        "scripts": {
          ...
          "test": "jest"
        },
        ...
        "jest": {
          "collectCoverageFrom": [
            "src/**/*.js"
          ],
          "moduleNameMapper": {
            "\\.(scss)$": "identity-obj-proxy"
          },
          "transform": {
            "^.+\\.js$": "babel-jest"
          }
        }
      }
      ```
    - Create `test/.eslintrc.json` to add some jest-specific rules:
      ```json
      {
        "extends": "mailonline/jest"
      }
      ```
    - Create `test/sagas/repos.spec.js`:
      ```js
      import {call, put} from 'redux-saga/effects';
      import {errorRequestingRepos, receivedRepos, requestRepos} from '../../src/actions/repos';
      import {fetchByOrg} from '../../src/api/repos';
      import {fetchReposFromApi} from '../../src/sagas/repos';

      describe('repo sagas', () => {
        describe('fetchReposFromApi', () => {
          it('puts receivedRepos action after fetching successfully', () => {
            const orgName = 'github';
            const requestAction = requestRepos(orgName);
            const mockResults = [
              {
                name: 'somerepo',
                url: 'http://github.com/github/somerepo'
              }
            ];
            const iterator = fetchReposFromApi(requestAction);

            expect(iterator.next().value).toEqual(call(fetchByOrg, orgName));
            expect(iterator.next(mockResults).value).toEqual(put(receivedRepos(mockResults)));
            expect(iterator.next().done).toBe(true);
          });

          it('puts error action if requesting repos failed', () => {
            const orgName = 'github';
            const requestAction = requestRepos(orgName);
            const error = new Error('error fetching repos');
            const iterator = fetchReposFromApi(requestAction);

            expect(iterator.next().value).toEqual(call(fetchByOrg, orgName));
            expect(iterator.throw(error).value).toEqual(put(errorRequestingRepos(error)));
            expect(iterator.next().done).toBe(true);
          });
        });
      });
      ```
    - Create `test/reducers/repos.spec.js`:
      ```js
      import {
        errorRequestingRepos,
        receivedRepos,
        requestRepos
      } from '../../src/actions/repos';
      import reposReducer, {initialState} from '../../src/reducers/repos';

      describe('repos reducer', () => {
        const mockRepos = [
          {
            name: 'reponame',
            url: 'http://github.com/org/reponame'
          }
        ];

        it('error action sets error', () => {
          const error = new Error('error fetching');
          const errorAction = errorRequestingRepos(error);

          expect(reposReducer(initialState, errorAction).error).toBe(error);
        });

        it('receivedRepos sets repos and stops loading', () => {
          const initialStateWithLoading = {
            ...initialState,
            loading: true
          };
          const receiveAction = receivedRepos(mockRepos);
          const newState = reposReducer(initialStateWithLoading, receiveAction);

          expect(newState.loading).toBe(false);
          expect(newState.repos).toBe(mockRepos);
        });

        it('requestRepos starts loading and empties results', () => {
          const orgName = 'github';
          const stateWithResults = {
            ...initialState,
            loading: true,
            repos: mockRepos
          };
          const requestAction = requestRepos(orgName);
          const newState = reposReducer(stateWithResults, requestAction);

          expect(newState.loading).toBe(true);
          expect(newState.repos).toBe(null);
        });
      });
      ```
    - Create `test/components/GetReposButton.spec.js` (using enzyme):
      ```js
      import React from 'react';
      import {shallow} from 'enzyme';
      import {GetReposButtonPureComponent as GetReposButton} from '../../src/components/GetReposButton';
      import {requestRepos} from '../../src/actions/repos';

      const HARDCODED_ORG_NAME = 'github';

      describe('<GetReposButton/>', () => {
        it('disables button while fetching', () => {
          const dispatch = jest.fn();
          const component = shallow(
            <GetReposButton dispatch={dispatch} fetching={true} />
          );

          expect(component.find('button').prop('disabled')).toBe(true);
          component.simulate('click');
          expect(dispatch).not.toHaveBeenCalled();
        });

        it('dispatches requestRepos on click when not disabled', () => {
          const dispatch = jest.fn();
          const component = shallow(
            <GetReposButton dispatch={dispatch} fetching={false} />
          );

          expect(component.find('button').prop('disabled')).toBe(false);
          component.simulate('click');
          expect(dispatch).toHaveBeenCalledTimes(1);
          expect(dispatch).toHaveBeenCalledWith(requestRepos(HARDCODED_ORG_NAME));
        });
      });
      ```
    - Create `test/components/RepoList.spec.js` (using snapshots):
      ```js
      import React from 'react';
      import renderer from 'react-test-renderer';
      import {RepoListPureComponent as RepoList} from '../../src/components/RepoList';

      const mockRepos = [
        {
          name: 'firstrepo',
          url: 'https://github.com/orgname/firstrepo'
        },
        {
          name: 'secondrepo',
          url: 'https://github.com/orgname/secondrepo'
        }
      ];

      describe('<RepoList/>', () => {
        it('renders "Loading" while fetching', () => {
          const tree = renderer.create(<RepoList fetching={true} />).toJSON();

          expect(tree).toMatchSnapshot();
        });

        it('renders "No results" when repos prop is null', () => {
          const tree = renderer.create(<RepoList fetching={false} repos={null} />).toJSON();

          expect(tree).toMatchSnapshot();
        });

        it('renders repo list with results when not fetching', () => {
          const tree = renderer.create(<RepoList fetching={false} repos={mockRepos} />).toJSON();

          expect(tree).toMatchSnapshot();
        });
      });
      ```
    - Get a coverage report with `npm run test -- --coverage`


  That's it! I hope you enjoyed the workshop. Feel free to contact me if you have any questions, or create an issue in this repository.
