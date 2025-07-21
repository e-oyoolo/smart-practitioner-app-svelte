<script lang="ts">
  import axios from "axios";
  import { onMount } from "svelte";
  import type { Bundle, OperationOutcome, BundleEntry, Observation } from "fhir/r4";
  import { formatRelative } from "date-fns";

  export let accessToken: string
  export let patientId: string
  export let baseUrl: string
  export let title: string = 'Observations'

  let temperature: number
  let obervationPosting = false

  onMount(async ()=>{

  })

  const getLabResults = async () =>{
    const labObservationResponse = await axios.get<Bundle<Observation | OperationOutcome>>(`${baseUrl}/Observation`, {
      params:{
        patient: patientId,
        sort:'-date',
        category:'vital-signs'
      },
      headers:{
        'Authorization': `Bearer ${ accessToken }`
      }
    })
    return labObservationResponse.data
  }

  let observationPromise = getLabResults()

  const postTemperature = async (temperature: number) =>{
    obervationPosting = true
    const temperatureResource = {
        "resourceType": "Observation",
        "status": "final",
        "category": [{
                "coding": [{
                        "system": "http://terminology.hl7.org/CodeSystem/observation-category",
                        "code": "vital-signs",
                        "display": "Vital Signs"
                    }
                ],
                "text": "Vital Signs"
            }
        ],
        "code": {
            "coding": [{
                    "system": "http://loinc.org",
                    "code": "8331-1"
                }
            ],
            "text": "Temperature Oral"
        },
        "subject": {
            "reference": `Patient/${patientId}`
        },
        "effectiveDateTime": new Date().toISOString(),
        "valueQuantity": {
            "value": temperature,
            "unit": "degC",
            "system": "http://unitsofmeasure.org",
            "code": "Cel"
        }
    }

    console.log(temperatureResource)
    
    const temperatureObservationResponse = await axios.post(`${baseUrl}/Observation`, temperatureResource,{
      headers:{
        'Authorization': `Bearer ${ accessToken }`
      }
    })
    obervationPosting = false
    console.log({temperatureObservationResponse})
    observationPromise = getLabResults()
  }  

  const getObservationDisplay = (observation: Observation | undefined) => {

    if(!observation){
      return ''
    }
    
    const codableConcept = observation?.valueCodeableConcept

    if(codableConcept){
      return observation?.valueCodeableConcept?.text
    }
    // const isBp = observation.code?.coding?.findIndex(a=>a.code === '55284-4')
    const isBp = observation.component != undefined

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
      const nonPanelResults = results.filter(entry=>entry?.resource?.hasMember === undefined)

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
<div class="mt-10 max-w-xl mx-auto">
  {#await observationPromise}
    Loading...
  {:then observations}
    <h1 class="text-2xl">
      { title }
    </h1>
    <div class="my-4">
      <p class="font-semibold text-gray-700">Create new temperature (degree Celsius)</p>
      <form on:submit|preventDefault={()=>{postTemperature(temperature)}}>
        <div>
        <input step="0.1" min=10 max=50 bind:value={temperature} type="number" class="border border-black p-2 w-48"/>
        {#if obervationPosting}
          <p class="p-2">
            creating observation...
          </p>
        {:else}
          <button class="bg-black text-white p-2" type="submit">Submit</button>
        {/if}
        </div>
      </form>
    </div>
    {#each getObservationEntries(observations) as observation, i}
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