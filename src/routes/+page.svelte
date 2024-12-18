<script>
    import { onMount } from 'svelte';
    
    let username = '';
    let password = '';
    let errorMessage = '';
    let showError = false;
    let authState = 'loading'; // loading, auth_required, no_auth_required, authorized
    
    // Get URL parameters helper
    function getURLParams() {
      const params = new URLSearchParams(window.location.search);
      return Object.fromEntries(params);
    }
    
    // Handle regular login
    async function handleLogin() {
      try {
        showError = false;
        const response = await fetch(`/api/captiveportal/access/logon/${window.zoneid}/`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({ user: username, password }),
        });
        
        const data = await response.json();
        
        if (data.clientState === 'AUTHORIZED') {
          const params = getURLParams();
          if (params.redirurl) {
            window.location.href = `http://${params.redirurl}?refresh`;
          } else {
            window.location.reload();
          }
        } else {
          username = '';
          password = '';
          errorMessage = 'Authentication failed';
          showError = true;
        }
      } catch (error) {
        errorMessage = 'Unable to connect to authentication server';
        showError = true;
      }
    }
    
    // Handle anonymous login
    async function handleAnonymousLogin() {
      try {
        showError = false;
        const response = await fetch(`/api/captiveportal/access/logon/${window.zoneid}/`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({ user: '', password: '' }),
        });
        
        const data = await response.json();
        
        if (data.clientState === 'AUTHORIZED') {
          const params = getURLParams();
          if (params.redirurl) {
            window.location.href = `http://${params.redirurl}?refresh`;
          } else {
            window.location.reload();
          }
        } else {
          errorMessage = 'Login failed';
          showError = true;
        }
      } catch (error) {
        errorMessage = 'Unable to connect to authentication server';
        showError = true;
      }
    }
    
    // Handle logout
    async function handleLogout() {
      try {
        showError = false;
        await fetch(`/api/captiveportal/access/logoff/${window.zoneid}/`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({ user: '', password: '' }),
        });
        window.location.reload();
      } catch (error) {
        errorMessage = 'Unable to connect to authentication server';
        showError = true;
      }
    }
    
    // Check initial auth status
    onMount(async () => {
      try {
        const response = await fetch(`/api/captiveportal/access/status/${window.zoneid}/`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({ user: '', password: '' }),
        });
        
        const data = await response.json();
        
        if (data.clientState === 'AUTHORIZED') {
          authState = 'authorized';
        } else if (data.authType === 'none') {
          authState = 'no_auth_required';
        } else {
          authState = 'auth_required';
        }
      } catch (error) {
        errorMessage = 'Unable to connect to authentication server';
        showError = true;
        authState = 'auth_required';
      }
    });
  </script>
  
  <svelte:head>
    <title>Captive Portal</title>
  </svelte:head>
    
    <main class="flex-1 flex items-center justify-center px-4 sm:px-6 lg:px-8">
        <div class="max-w-md w-full space-y-8">
        <div class="bg-white py-8 px-4 shadow sm:rounded-lg sm:px-10">
            {#if authState === 'loading'}
            <div class="text-center">
                <h2 class="text-xl font-medium text-gray-900">Loading...</h2>
            </div>
            {:else if authState === 'auth_required'}
            <div>
                <h2 class="text-center text-xl font-bold text-gray-900 mb-8">Please sign in</h2>
                <form on:submit|preventDefault={handleLogin} class="space-y-6">
                <div>
                    <input
                    type="text"
                    bind:value={username}
                    placeholder="Username"
                    required
                    autocomplete="username"
                    autocapitalize="none"
                    class="appearance-none block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-blue-500 focus:border-blue-500"
                    />
                </div>
                <div>
                    <input
                    type="password"
                    bind:value={password}
                    placeholder="Password"
                    required
                    autocomplete="current-password"
                    class="appearance-none block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-blue-500 focus:border-blue-500"
                    />
                </div>
                <button
                    type="submit"
                    class="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500"
                >
                    Sign in
                </button>
                </form>
            </div>
            {:else if authState === 'no_auth_required'}
            <div class="text-center">
                <h2 class="text-xl font-bold text-gray-900 mb-8">Welcome</h2>
                <button
                on:click={handleAnonymousLogin}
                class="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500"
                >
                Continue to Internet
                </button>
            </div>
            {:else}
            <div class="text-center">
                <h2 class="text-xl font-bold text-gray-900 mb-8">You are connected</h2>
                <button
                on:click={handleLogout}
                class="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500"
                >
                Logout
                </button>
            </div>
            {/if}
        </div>
        </div>
    </main>

    {#if showError}
        <div class="fixed bottom-4 right-4 max-w-md bg-red-50 p-4 rounded-md">
        <div class="flex">
            <div class="flex-shrink-0">
            <svg class="h-5 w-5 text-red-400" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd" />
            </svg>
            </div>
            <div class="ml-3">
            <p class="text-sm font-medium text-red-800">
                {errorMessage}
            </p>
            </div>
        </div>
        </div>
    {/if}