<script lang="ts">
    import axios from "axios";
    import { onMount } from "svelte";

    let iss: string
    let launch: string
    let authorizationEndpoint: string
    let redirectUrl: string

    const constructAuthUrl = (launch: string) => {
        const url = new URL(authorizationEndpoint)
        url.searchParams.set('client_id','5d21112e-1d1d-4ceb-9018-6741ebcddc80')
        url.searchParams.set('redirect_uri','http://localhost:5173/')
        url.searchParams.set('scope','openid fhirUser launch user/Observation.read user/Observation.write user/Patient.read')
        url.searchParams.set('response_type','code')
        url.searchParams.set('aud',iss)

        url.searchParams.set('launch', launch)

        return url.href
    }
    onMount(async()=>{
        const launchUrl = new URL(window.location.href)
        const issParam = launchUrl.searchParams.get("iss")
        const launchParam = launchUrl.searchParams.get("launch")

        if(!issParam || !launchParam){
            throw new Error("iss or launch parameters not found")
        }

        iss = issParam
        launch = launchParam

        const smartConfigurationResponse = await axios.get(`${iss}/.well-known/smart-configuration`)
        const smartConfiguration = smartConfigurationResponse.data
        authorizationEndpoint = smartConfiguration.authorization_endpoint as string

        redirectUrl = constructAuthUrl(launch)
        console.log(redirectUrl)

        window.location.href = redirectUrl
    })
</script>

<div>
    <pre>
        { JSON.stringify({iss, launch, authorizationEndpoint, redirectUrl}, null, 2)}
    </pre>
</div>