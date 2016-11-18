#vuefire-auth-demo

## Barebones authentication demo with Vue and VueFire

note: Vue 2.x

Demonstrates basic anonymous authentication and a pattern that can be used when a Firebase path is dependent on async operations that must complete prior to Firebase binding (e.g. `/messages/${user.uid}` requires `user.uid` to be known).

```bash
npm install
npm run dev
```

Or see the live demo: [http://vuefire-auth-example.firebaseapp.com](https://vuefire-auth-example.firebaseapp.com/)

### Details

The path that the demonstration requires to access data for a session is `/messages/${uid}` - where `${uid}` is the (anonymous) Firebase `user.uid`. Note: this database structure pattern is illustrated in the [Firebase docs](https://firebase.google.com/docs/database/web/structure-data) at the time of this writing.

Since we don't have a `user` prior to authentication, we cannot evaluate the path to give to VueFire.  
 
So essentially what we do is:
   
   * Initiate authentication as early as possible. In this demo, we're using [anonymous authentication](https://firebase.google.com/docs/reference/js/firebase.auth.Auth#signInAnonymously) so there is no need to consider user input (i.e. credentials):

```javascript
new Vue({
  el: '#app',
  beforeCreate: function() {
    // Our earliest lifecycle hook and first access
    // to $bindAsArray() / $bindAsObject() from VueFire
    // Start Firebase authentication here:
    firebase.auth().onAuthStateChanged(function(user) {
        // Once authed, we'll have a user.uid and
        // can bind references that require it
        ...  
```

   * Add placeholder ([reactive](https://vuejs.org/v2/guide/reactivity.html#How-Changes-Are-Tracked)) properties on `data` that will house the eventual Firebase references. This allows the UI to render immediately (sans missing data) and the reactivity handles UI updates once the data is available:
    
```javascript
data: {
    user: {},
    messages: []
}
```

   * Once authed, bind the `data` properties to Firebase references:

```javascript
firebase.auth().onAuthStateChanged(function(user) {
  if (user) {
    this.user = user
    this.$bindAsArray('messages', db.ref(`messages/' + user.uid))
  } else {
    firebase.auth().signInAnonymously().catch(console.error)
  }
}.bind(this))
```

***

Initially created based on [this thread](https://github.com/vuejs/vuefire/issues/47).
