<script lang="ts">
    import SessionKit, { BrowserLocalStorage, Session } from '@wharfkit/session'
    import { WalletPluginAnchor } from '@wharfkit/wallet-plugin-anchor'
    import { WalletPluginCloudWallet } from '@wharfkit/wallet-plugin-cloudwallet'
    import WebRenderer from '@wharfkit/web-renderer'
    import { browser } from '$app/environment'
    import { onMount } from 'svelte'

    let session: Session | undefined
    let sessionKit: SessionKit

    if (browser) {
        const ui = new WebRenderer()

        sessionKit = new SessionKit({
            appName: 'demo',
            chains: [
                {
                    id: 'aca376f206b8fc25a6ed44dbdc66547c36c6c33e3a119ffbeaef943642f0e906',
                    url: 'https://eos.greymass.com',
                },
                {
                    id: '4667b205c6838ef70ff7988f6e8257e8be0e1284a2f59699054a018f743b1d11',
                    url: 'https://telos.greymass.com',
                },
                {
                    id: '8fc6dce7942189f842170de953932b1f66693ad3788f766e777b6f9d22335c02',
                    url: 'https://api.uxnetwork.io',
                },
                {
                    id: '1064487b3cd1a897ce03ae5b6a865651747e2e152090f99c1d19d44e01aea5a4',
                    url: 'https://wax.greymass.com',
                },
            ],
            storage: new BrowserLocalStorage('demo'),
            ui,
            walletPlugins: [
                new WalletPluginAnchor(),
                new WalletPluginCloudWallet(),
            ],
        })

        onMount(async () => {
            session = await sessionKit.restore()
        })
    }

    async function login() {
        const response = await sessionKit.login()
        session = response.session
    }

    async function logout() {
        await sessionKit.logout(session)
        session = undefined
    }

    async function transact() {
        if (!session) {
            throw new Error('cannot transact without a session')
        }
        const action = {
            account: 'eosio.token',
            name: 'transfer',
            authorization: [session.permissionLevel],
            data: {
                from: session.actor,
                to: 'teamgreymass',
                quantity: '0.00000001 WAX',
                memo: 'Yay WharfKit! Thank you <3',
            },
        }
        session.transact({ action }, { broadcast: false }).catch((e) => {
            console.log('error caught in transact', e)
        })
    }
</script>

<div class="login">
    {#if session}
        <div>
            <p>Logged in as {session.actor}</p>
            <button type="button" class="primary" on:click={transact}
                >Test Transaction (No Broadcast)</button
            >
            <button type="button" on:click={logout}>Logout</button>
        </div>
    {:else}
        <div>
            <button type="button" on:click={login}>Login</button>
        </div>
    {/if}
</div>

<style>
</style>
