# Self Service

Create a self service portal by creating a HTML page in [Binaries](../beginning/binaries.md) under `selfservice/index.html`. The self service portal can be reached by own users under https://project.realmigrator.com/selfservice[^1]

A minimal working sample can be found below:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>realmigrator Self Service Portal</title>
  <link rel="stylesheet" href="https://unpkg.com/purecss@1.0.0/build/pure-min.css" integrity="sha384-nn4HPE8lTHyVtfCBi5yW9d20FjT8BJwUXyWZT9InLYax14RDjBj46LmSztkmNP9w" crossorigin="anonymous">
</head>
<body>
  <div class="pure-g">
    <div class="pure-u-1-5"></div>
    <div class="pure-u-3-5">
      <h1>realmigrator Self Service Portal</h1>
    </div>
    <div class="pure-u-1-5"></div>
  </div>
  <div class="pure-g">
    <div class="pure-u-1-5"></div>
    <div class="pure-u-3-5" id="app">
    </div>
    <div class="pure-u-1-5"></div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/preact/dist/preact.min.js"></script>
  <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <script type="text/babel">
    /** @jsx h */
const { Component, h, render } = window.preact;
class App extends Component {
  componentDidMount() {
    const urlParams = new URLSearchParams(window.location.search);
    const error = urlParams.get('error');
    if(error) {
      this.setState({ error: error });
    } else {
      axios.get('/selfservice/api/userinfo').then((response) => {
        this.setState({ info: response.data });
      }).catch((error) => {
        this.setState({ error: true });
      });
    }
  }
  render(props, state) {
    if(state.error) {
      if(typeof state.error === 'string') {
        return (
          <span style={ { color: 'red' } } >{state.error}</span>
        );
      } else {
        return (
          <a className="pure-button pure-button-primary" href="/selfservice/login">Login</a>
        );
      }
    } else if(state.info) {
      const js = JSON.stringify(state.info, null, 2);
      return (
        <form className="pure-form">
          <textarea className="pure-input-1" placeholder="no data found">{js}</textarea>
        </form>
      );
    }
    return (
      <div/>
    );
  }
}
render(<App/>, document.getElementById('app'));
</script>
</body>
</html>
```

[^1]: Use an actual project name instead of 'project'
