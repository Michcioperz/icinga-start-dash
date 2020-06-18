<script>
  export let item;
  let host;
  let service;
  const states = [
    {name: "OK", clazz: "ok"},
    {name: "WARNING", clazz: "warning"},
    {name: "CRITICAL", clazz: "critical"},
    {name: "UNKNOWN", clazz: "unknown"},
  ];
  $: if (item.host_display_name != item.host_name) {
    host = item.host_display_name + " (" + item.host_name + ")";
  } else {
    host = item.host_display_name;
  }
  $: if (item.service_description != item.service_display_name) {
    service = item.service_description + " (" + item.service_display_name + ")";
  } else {
    service = item.service_description;
  }
  $: state = states[item.service_state];
</script>

<aside>
  <div class="owo {state.clazz}"></div>
  <h3>{host}</h3>
  <h4>{service}</h4>
  <a href="https://seraph.dasie.mimuw.edu.pl/monitoring/service/show?host={item.host_name}&service={item.service_description}" target="_blank">See in Icinga</a>
  <slot></slot>
  <pre>{item.service_output}</pre>
</aside>

<style>
  .owo {
    width: 5em;
    height: 5em;
  }
  pre {
   white-space: pre-wrap;
  }
</style>
