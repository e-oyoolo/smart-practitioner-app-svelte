<script lang="ts">
  import axios from "axios";
  import { formatRelative, subDays } from 'date-fns';
  import type {Bundle, BundleEntry, MedicationRequest, Observation, OperationOutcome } from 'fhir/r4'

  export let accessToken:string;
  export let patientId: string;
  export let baseUrl: string;
  export let title: string = 'Observations'

  const getLabResults = async () =>{
    const labObservationRequestResponse = await axios.get<Bundle<Observation|OperationOutcome>>(`${baseUrl}/Observation`,{
      params:{
        patient:patientId,
        // category,
        sort:'-date'
      },
      headers:{
        'Authorization': `Bearer ${accessToken}`
      }
    })

    return labObservationRequestResponse.data
  }

  const getObservationDisplay = (observation: Observation) => {
    const isBp = observation.code?.coding?.find(a=>a.code === '55284-4')
  }
  // const getObservationEntries = (bundle: Bundle<Observation | OperationOutcome>): BundleEntry<Observation>[] =>{
  //   const results = bundle.entry?.filter((entry)=> entry.resource?.resourceType === 'Observation') as BundleEntry<Observation> || []

  //   return results.sort((a,b) =>{
  //     if(a?.resource?.effectiveDateTime && b?.resource?.effectiveDateTime){

  //     }
  //   })
  // }
</script>
<main>
</main>