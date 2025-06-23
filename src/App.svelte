<a href=/db>DB</a>
<a href=/times>Times</a>

<script>
import {Route, router} from 'tinro'
import Db from './Db.svelte'
// import Times from './Times.svelte'
import Times from './TimesSimple.svelte'

router.mode.hash()
</script>

<Route>
  <Route path=/db><Db/></Route>
  <Route path=/times><Times/></Route>
</Route>
