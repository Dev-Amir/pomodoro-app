<script>
  const electron = require("electron").remote;
  import Router from "svelte-spa-router";
  import { tweened } from "svelte/motion";
  import { quadOut } from "svelte/easing";

  import Dashboard from "./pages/Dashboard.svelte";
  import { Modal } from "./components";
  import { storage } from "./utils";
  import { store } from "./store";

  const visible = tweened(0, {
    duration: 250,
    easing: quadOut
  });
  const routes = {
    "/": Dashboard
  };
  electron.getCurrentWindow().on("show", () => {
    visible.set(1);
  });
  electron.getCurrentWindow().on("animateHidden", () => {
    visible.set(0);
    setTimeout(() => {
      electron.getCurrentWindow().hide();
    }, 250);
  });
  storage.load({ key: "appStorage" }).then(data => {
    if (data && Object.values(data).length) store.changeAppData(data);
    store.subscribe(data => {
      storage.save({ key: "appStorage", data });
    });
  });
</script>

<style>
  #app {
    display: flex;
    background: var(--color-first);
    flex: 1;
    position: relative;
    border-radius: 0.5rem;
  }
  #app.platform-darwin {
    margin: 1rem 2rem 2rem;
    box-shadow: 0 0.1rem 2rem rgba(0, 0, 0, 0.2);
  }
  #app.platform-darwin::before {
    content: "";
    width: 0;
    height: 0;
    border-left: 0.5rem solid transparent;
    border-right: 0.5rem solid transparent;
    border-bottom: 0.5rem solid var(--color-first);
    position: absolute;
    top: -0.5rem;
    left: 50%;
    transform: translateX(-50%);
  }
</style>

<div
  id="app"
  class={`platform-${process.platform}`}
  style="transform: translateY({10 - $visible * 10}px); opacity: {$visible}">
  <Router {routes} />
  <Modal />
</div>
