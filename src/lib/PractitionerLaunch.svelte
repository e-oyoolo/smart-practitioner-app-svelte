<script lang="ts">
  import axios from "axios";
  import { stringify } from "postcss";
  import { onMount } from "svelte";
  import PatientBanner from "./PatientBanner.svelte";
  import ObservationViewer from "./ObservationViewer.svelte";

  let clientId = "5d21112e-1d1d-4ceb-9018-6741ebcddc80";
  let redirectUri = "http://localhost:5173/";
  let baseUrl: string;

  interface TokenResponse {
    access_token: string;
    patient: string;
    scope: string;
    need_patient_banner: boolean;
    id_token: string;
    smart_style_url: string;
    expires_in: number;
  }

  let token: TokenResponse | undefined = undefined;

  const LOCAL_STORAGE_TOKEN_ENDPOINT = "tokenEndpoint";
  const LOCAL_STORAGE_TOKEN_JSON = "token";
  const LOCAL_STORAGE_ISS = "iss";


  const constructAuthUrl = (
    authorizationEndpoint: string,
    launch: string,
    iss: string
  ) => {
    const url = new URL(authorizationEndpoint);
    url.searchParams.set("client_id", clientId);
    url.searchParams.set("redirect_uri", redirectUri);
    url.searchParams.set(
      "scope",
      "openid fhirUser launch user/Observation.read user/Observation.write user/Patient.read"
    );
    url.searchParams.set("response_type", "code");
    url.searchParams.set("aud", iss);

    url.searchParams.set("launch", launch);

    return url.href;
  };

  const makeTokenRequest = async (code: string) => {
    const tokenEndpoint = localStorage.getItem(LOCAL_STORAGE_TOKEN_ENDPOINT);

    if (!tokenEndpoint) {
      throw new Error("Token endpoint could not be found in local storage");
    }

    const form = new URLSearchParams();
    form.set("grant_type", "authorization_code");
    // form.set('grant_type', 'authorization_code')
    form.set("code", code);
    form.set("redirect_uri", redirectUri);
    form.set("client_id", clientId);

    const tokenResponse = await axios.post(tokenEndpoint, form, {
      headers: {
        "Content-Type": "application/x-www-form-urlencoded",
      },
    });

    localStorage.removeItem(LOCAL_STORAGE_TOKEN_ENDPOINT);

    return tokenResponse.data;
  };

  onMount(async () => {
    const launchUrl = new URL(window.location.href);
    const issParam = launchUrl.searchParams.get("iss");
    const launchParam = launchUrl.searchParams.get("launch");

    const code = launchUrl.searchParams.get("code");

    const tokenJSON = localStorage.getItem(LOCAL_STORAGE_TOKEN_JSON);
    const issLocalStorage = localStorage.getItem(LOCAL_STORAGE_ISS);

    if (issLocalStorage) {
      baseUrl = issLocalStorage;
    }

    // if (tokenJSON) {
    //   token = JSON.parse(tokenJSON);
    //   return;
    // }

    if (token) {
      //validate token expiry
      return;
    }

    if (code) {
      const tokenFromCerner = await makeTokenRequest(code);
      // console.log({ tokenFromCerner });
      token = tokenFromCerner;

      localStorage.setItem(LOCAL_STORAGE_TOKEN_JSON, JSON.stringify(token));
      return;
    }

    if (!issParam || !launchParam) {
      throw new Error("iss or launch parameters not found");
    }

    const iss = issParam;
    localStorage.setItem(LOCAL_STORAGE_ISS, issParam);
    const launch = launchParam;

    const smartConfigurationResponse = await axios.get(
      `${iss}/.well-known/smart-configuration`
    );
    const smartConfiguration = smartConfigurationResponse.data;
    const authorizationEndpoint =
      smartConfiguration.authorization_endpoint as string;
    const tokenEndpoint = smartConfiguration.token_endpoint as string;

    localStorage.setItem(LOCAL_STORAGE_TOKEN_ENDPOINT, tokenEndpoint);

    const redirectUrl = constructAuthUrl(authorizationEndpoint, launch, iss);

    window.location.href = redirectUrl;
  });
</script>

{#if !token}
  Loading...
{:else if token?.need_patient_banner}
  <PatientBanner {baseUrl} accessToken={token.access_token} patientId={token.patient}></PatientBanner>
  <ObservationViewer { baseUrl } accessToken={token.access_token} patientId={token.patient}></ObservationViewer>
{/if}
