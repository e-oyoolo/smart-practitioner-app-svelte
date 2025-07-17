<script lang="ts">
  import axios from "axios";
  import { onMount } from "svelte";
  import { FHIR_BASE_URL } from '../config'
  import type { MedicationRequest, Patient, Bundle, OperationOutcome, BundleEntry } from "fhir/r4";
  import { stringify } from "postcss";

  export let accessToken: string
  export let patientId: string

  onMount(async ()=>{

  })

  const getMedications = async () =>{
    const medicationRequestResponse = await axios.get<Bundle<MedicationRequest | OperationOutcome>>(`${FHIR_BASE_URL}/MedicationRequest`, {
      params:{
        subject: patientId
      },
      headers:{
        'Authorization': `Bearer ${ accessToken }`
      }
    })
    return medicationRequestResponse.data
  }

  const getMedicationRequestEntries = (bundle: Bundle<MedicationRequest | OperationOutcome>): BundleEntry<MedicationRequest>[] =>{
    if(!bundle?.entry){
      return []
    }
    return bundle.entry?.filter(entry=>entry.resource?.resourceType === 'MedicationRequest') as BundleEntry<MedicationRequest>[]
  }

</script>
<div class="mt-10 max-width-md mx-auto">
  {#await getMedications()}
    Loading...
  {:then medicationList}
    <h1 class="text-2xl">
      Medication List
    </h1>
    {#each getMedicationRequestEntries(medicationList) as medication, i}
      <p class="font-medium">
        {i + 1}.{medication.resource?.medicationReference?.display}
      </p>
      <div class="ml-4">
        {#if medication?.resource?.reasonCode?.[0]?.text}
          <p>
            Medication for {medication?.resource?.reasonCode?.[0]?.text}
          </p>
        {/if}
        {#if medication?.resource?.dosageInstruction?.[0]?.patientInstruction} 
          <p>
            Dosage: {medication?.resource?.dosageInstruction?.[0]?.patientInstruction}
          </p>
        {/if}
      </div>
    {/each}
  {/await}
</div>