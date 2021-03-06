# hoc-enzyme
Solving painful headless testing of Relay-produced HOCs

Install
-------

1. npm install -D hoc-enzyme enzyme
2. Edit your Mocha's setup.js
  1. import HOCEnzyme from 'hoc-enzyme'
  2. HOCEnzyme.setup()

Syntax
======
```javascript
wrapper.diveInto((child) => child.type() == Text)
```

With DOM rendering (mounted components)
=======================================
Usage with Redux && react-native-router-flux
--------------------------------------------
```javascript
describe('Sandboxed react-native-router-flux width redux', () => {
  let wrapper

  jsdom()

  before(() => {
    wrapper = mount(<RouterComponent />)
  })

  it.only('should switch between views', () => {
    let spy = sinon.spy(Actions, 'second')
    const TmpCom = wrapper.find(RouterWithRedux).diveInto(
      child => {
        return child.type() == Scene && child.__unwrapped.key == 'home'
      }
    ).prop('component')
    expect(store.getState()).to.have.deep.property('scene.sceneKey', 'home')
    mount(<TmpCom />).diveInto(
      child => child.type() == TouchableHighlight
    ).prop('onPress')()
    expect(store.getState()).to.have.deep.property('scene.sceneKey', 'second')
  })
})
```

Usage with Apollo
-----------------
```javascript
describe('Sandboxed react-native + apollo', () => {
  let wrapper

  jsdom()

  before(() => {
    wrapper = mount((
      <ApolloProvider client={client}>
        <SandboxComponent />
      </ApolloProvider>
    ))
  })

  it('should fetch data from apollo component', () => {
    const tmp = wrapper.find(SandboxComponent).diveInto((child) => {
      return child.type() == Text
    }, false)

    expect(tmp.first()).to.have.props({children: 'Hello world'})
  })
})
```

Usage with Relay
----------------
```javascript
describe('<Home/>', () => {
  let wrapper = null

  jsdom()

  const counter = (child, node, index) =>
    node.type() == View && child.type() == Text && index == 2

  const button = child => child.type() == TouchableHighlight

  before(done => {
    /**
     * Home is a wrapped with Renderer Relay Container
     * It renders the following jsx
     * <View>
     *   <TouchableHighlight ...>
     *     <Text>...</Text>
     *   </TouchableHighlight>
     *   <Text>...</Text>
     *   <Text>...</Text>
     * </View>
     */
    wrapper = mount(<Home />)
    done()
  })

  it('should render select text', done => {
    const view = wrapper.find(View)
    // Use predicate to find not presented children
    expect(view.diveInto(counter)).to.have.props({children: 0})

    view.diveInto(button).prop('onPress')()
    setTimeout(() => {
      expect(view.diveInto(counter)).to.have.props({children: 1})
      done()
    }, 10)
  })
})
```
