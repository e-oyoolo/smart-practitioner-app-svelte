<script lang="ts">
  import axios from "axios";
  import { onMount } from "svelte";
  import type { Bundle, OperationOutcome, BundleEntry, Observation } from "fhir/r4";
  import { formatRelative } from "date-fns";

  export let accessToken: string
  export let patientId: string
  export let baseUrl: string
  export let title: string = 'Observations'

  onMount(async ()=>{

  })

  const getLabResults = async () =>{
    const labObservationResponse = await axios.get<Bundle<Observation | OperationOutcome>>(`${baseUrl}/Observation`, {
      params:{
        patient: patientId,
        sort:'-date'
      },
      headers:{
        'Authorization': `Bearer ${ accessToken }`
      }
    })

    console.log('labObservationResponse.data')
    console.log(labObservationResponse.data)
    console.log('labObservationResponse.request')
    console.log(labObservationResponse.request)
    return labObservationResponse.data
  }

  const getObservationDisplay = (observation: Observation | undefined) => {

    if(!observation){
      return ''
    }
    
    const isBp = observation.code?.coding?.findIndex(a=>a.code === '55284-4')

    if(isBp){
      const systolicComponent = observation.component?.find(a=>a.code?.coding?.find(b=>b.code === '8480-6'))
      const systolic = systolicComponent?.valueQuantity?.value
      const diastolicComponent = observation.component?.find(a=>a.code?.coding?.find(b=>b.code === '8462-4'))
      const diastolic = diastolicComponent?.valueQuantity?.value

      return `${systolic}/${diastolic}`
    }

    if(!observation?.valueQuantity?.unit){
      return observation?.valueQuantity?.value
    }

    return `${observation?.valueQuantity?.value} ${observation?.valueQuantity?.unit}`
  }
  const getObservationEntries = (bundle: Bundle<Observation | OperationOutcome>): BundleEntry<Observation>[] =>{
    if(!bundle?.entry){
      return []
    }

    const results = bundle.entry?.filter(entry=>entry.resource?.resourceType === 'Observation') as BundleEntry<Observation>[]
      const nonPanelResults = results.filter(entry=>!entry.resource?.hasMember)

    // results.sort((entry1,entry2)=>(entry1.resource?.effectiveDateTime < entry2.resource?.effectiveDateTime) ? -1:((entry1.resource?.effectiveDateTime > entry2.resource?.effectiveDateTime) ? 1:0))
    return nonPanelResults
    // .sort((entry1, entry2)=>{
    //   if(entry1?.resource?.effectiveDateTime && entry2?.resource?.effectiveDateTime){
    //     return new Date(entry2?.resource?.effectiveDateTime).getTime() - new Date(entry1?.resource?.effectiveDateTime).getTime() 
    //   }

    //   return 0
    // })
  }

</script>
<div class="mt-10 max-width-md mx-auto">
  {#await getLabResults()}
    Loading...
  {:then labResults}
    <h1 class="text-2xl">
      { title }
    </h1>
    {#each getObservationEntries(labResults) as observation, i}
      <p class="font-medium">
        <span class="font-medium">
          {observation.resource?.code?.text}
          {#if observation?.resource?.effectiveDateTime}
            ({formatRelative(new Date(observation?.resource?.effectiveDateTime), new Date())})
          {/if} 
        </span>
        { getObservationDisplay(observation.resource)}
      </p>
      <div class="ml-4">

      </div>
    {/each}
  {/await}
</div>