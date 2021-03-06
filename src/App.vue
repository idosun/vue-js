<template>
  <div id="app">
      <div id="iconsDiv">
        <img id="sentryIcon" alt="Sentry logo" src="./assets/sentry-logo.png" />
        <p class="plus">+</p>
        <img class="icon" alt="Vue logo" src="./assets/logo.png" />
      </div>

      <p id="greeting">{{greetingTxt}}</p>
      <div id="email-div">
        <input id="emailInput" v-model="userEmail" placeholder="Enter email..." type="email" />
        <button class="event-button" v-on:click="submitEmail">Submit</button>
      </div>
      <div id="event-list">
        <EventButton buttonTitle="Handled [SyntaxError]" :onClick="handledError" />
        <EventButton buttonTitle="Unhandled [RangeError]" :onClick="unhandledError" />
        <EventButton buttonTitle="Capture Message" :onClick="captureMessage" />
        <EventButton buttonTitle="UnHandled Promise Rejection" :onClick="unhandledPromiseRejection" />
      </div>
    </div>
</template>

<script>
import EventButton from "./components/EventButton.vue";
import Vue from "vue";
import * as Sentry from "@sentry/browser";
import * as Integrations from "@sentry/integrations";

const HELLO = "Hello";

Sentry.init({
  dsn: "https://1bddec6f128949e3bd9de147c10447a1@sentry.io/1889218",
  release: process.env.VUE_APP_RELEASE,
  environment: "prod",
  integrations: [new Integrations.Vue({ Vue, attachProps: true }), new Integrations.RewriteFrames()],
  beforeSend: function(event, hint){
    debugger;
    return event;
  }
});

export default {
  name: "app",
  components: {
    EventButton
  },
  data: function() {
    return { greetingTxt: HELLO, userEmail: "" };
  },
  methods: {
    submitEmail: function() {
      Sentry.configureScope(scope => {
        scope.setUser({ email: this.userEmail });
      });

      var newGreeting = HELLO + " " + this.userEmail;
      this.$set(this.$data, "greetingTxt", newGreeting);
    },

    notAFunctionError: function() {
      var someArray = [{ func: function() {} }];
      someArray[1].func();
    },

    unhandledPromiseRejection: function(){
      Promise.reject(new Error('fail')).then(function(result) {
        console.log('Resolved');
      });
    },

    captureMessage: function() {
      //decodeURIComponent("%");
      Sentry.withScope(function(scope) {
        scope.setLevel('info');
        Sentry.captureMessage('Message Captured: This shouldnt happen');
      });
    },

    handledError: function() {
       try{
            eval("foo bar");
          }catch(error){
              Sentry.captureException(error);
          }
    },

    unhandledError: function() {
      throw new RangeError("Parameter must be between 1 and 100");
    }
  }
};
</script>

<style>

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 900px;

  background-image: url("./assets/sentry-pattern.png");
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
}

#email-div {
  width: 600px;
  display: inline-flex;
}

#emailInput {
  height: 30px;
  width: 90%;
}

#event-list {
  height: 400px;
  padding: 3rem;
}

#greeting {
  margin-bottom: 0.8rem;
  margin-top: -2rem;
  font-size: 2rem;
}

.icon {
  height: 6rem;
  width: 6rem;
  padding-top: 1.6rem;
  padding-left: 2rem;
}

#sentryIcon {
  height: 9rem;
  width: 9rem;
}

#iconsDiv {
  display: inline-flex;
  padding-bottom: 2rem;
}

.plus {
  font-size: 2.6rem;
}
</style>
