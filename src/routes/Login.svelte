<script lang="ts">
    import { Chains, Session, SessionKit } from '@wharfkit/session'
    import { WalletPluginAnchor } from '@wharfkit/wallet-plugin-anchor'
    import WebRenderer from '@wharfkit/web-renderer'
    import { browser } from '$app/environment'
    import { onMount } from 'svelte'

    let session: Session | undefined
    let sessionKit: SessionKit

    if (browser) {
        const ui = new WebRenderer()

        sessionKit = new SessionKit({
            appName: 'demo',
            chains: [Chains.Jungle4],
            ui,
            walletPlugins: [new WalletPluginAnchor()],
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
