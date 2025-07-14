<script lang="ts">
    import axios from "axios";
    import { onMount } from "svelte";

    let iss: string
    let launch: string
    // let authorizationEndpoint: string
    // let tokenEndpoint: string
    let redirectUrl: string

    const LOCAL_STORAGE_TOKEN_ENDPOINT = 'tokenEndpoint'

    const constructAuthUrl = (authorizationEndpoint:string, launch: string) => {
        const url = new URL(authorizationEndpoint)
        url.searchParams.set('client_id','5d21112e-1d1d-4ceb-9018-6741ebcddc80')
        url.searchParams.set('redirect_uri','http://localhost:5173/')
        url.searchParams.set('scope','openid fhirUser launch user/Observation.read user/Observation.write user/Patient.read')
        url.searchParams.set('response_type','code')
        url.searchParams.set('aud',iss)

        url.searchParams.set('launch', launch)

        return url.href
    }

    const makeTokenRequest = (code: string)=>{
        const tokenEndpoint = localStorage.getItem(LOCAL_STORAGE_TOKEN_ENDPOINT)

        if(!tokenEndpoint){
            throw new Error('Token endpoint could not be found in local storage')
        }

        const form = new FormData()

        
        axios.post(tokenEndpoint, {})

        localStorage.removeItem(LOCAL_STORAGE_TOKEN_ENDPOINT)
    }

    onMount(async()=>{
        const launchUrl = new URL(window.location.href)
        const issParam = launchUrl.searchParams.get("iss")
        const launchParam = launchUrl.searchParams.get("launch")

        const code = launchUrl.searchParams.get("code")

        if (code){

            return 
        }

        if(!issParam || !launchParam){
            throw new Error("iss or launch parameters not found")
        }

        iss = issParam
        launch = launchParam

        const smartConfigurationResponse = await axios.get(`${iss}/.well-known/smart-configuration`)
        const smartConfiguration = smartConfigurationResponse.data
        const authorizationEndpoint = smartConfiguration.authorization_endpoint as string
        const tokenEndpoint = smartConfiguration.token_endpoint as string

        localStorage.setItem(LOCAL_STORAGE_TOKEN_ENDPOINT, tokenEndpoint)

        redirectUrl = constructAuthUrl(authorizationEndpoint, launch)

        window.location.href = redirectUrl
    })
</script>

<div>
    <pre>
        Loading...
    </pre>
</div>