<script lang="ts">
  import axios from "axios";
  import type { Patient } from "fhir/r4";
  import { onMount } from "svelte";
  // import tailwindcss from "@tailwindcss/vite";

  export let accessToken: string;
  export let baseUrl: string;
  export let patientId: string;

  let patientResource: Patient | undefined = undefined;

  onMount(async () => {
    const patientResourceResponse = await axios.get<Patient>(
      `${baseUrl}/Patient/${patientId}`,
      { headers: { Authorization: `Bearer ${accessToken}` } }
    );
    console.log("baseurl");
    console.log(baseUrl);
    patientResource = patientResourceResponse.data;
  });
</script>

<div class="p-3 flex gap-5 bg-slate-800 text-white">
  {#if patientResource}
    <p>
      Name: {patientResource?.name?.[0]?.text}
    </p>
    <p>DOB: {patientResource?.birthDate}</p>
    <p>Gender: {patientResource?.gender}</p>
  {:else}
    Loading Patient...
  {/if}
</div>