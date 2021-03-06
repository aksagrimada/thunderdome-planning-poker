<script>
    import { onMount } from 'svelte'

    import DownCarrotIcon from '../components/icons/DownCarrotIcon.svelte'
    import PageLayout from '../components/PageLayout.svelte'
    import SolidButton from '../components/SolidButton.svelte'
    import WarriorAvatar from '../components/WarriorAvatar.svelte'
    import { warrior } from '../stores.js'
    import { validateName, validatePasswords } from '../validationUtils.js'
    import { _ } from '../i18n'

    export let xfetch
    export let router
    export let notifications
    export let eventTag

    let warriorProfile = {}

    let updatePassword = false
    let warriorPassword1 = ''
    let warriorPassword2 = ''

    const avatarService = appConfig.AvatarService
    let avatars

    if (avatarService == 'dicebear') {
        avatars = [
            'male',
            'female',
            'human',
            'identicon',
            'bottts',
            'avataaars',
            'jdenticon',
            'gridy',
            'code',
        ]
    } else if (avatarService == 'gravatar') {
        avatars = [
            'mp',
            'identicon',
            'monsterid',
            'wavatar',
            'retro',
            'robohash',
        ]
    } else if (avatarService == 'robohash') {
        avatars = ['set1', 'set2', 'set3', 'set4']
    }

    function toggleUpdatePassword() {
        updatePassword = !updatePassword
        eventTag(
            'update_password_toggle',
            'engagement',
            `update: ${updatePassword}`,
        )
    }

    xfetch(`/api/warrior/${$warrior.id}`)
        .then(res => res.json())
        .then(function(wp) {
            warriorProfile = wp
        })
        .catch(function(error) {
            notifications.danger($_('pages.warriorProfile.errorRetreiving'))
            eventTag('fetch_profile', 'engagement', 'failure')
        })

    function updateWarriorProfile(e) {
        e.preventDefault()
        const body = {
            warriorName: warriorProfile.name,
            warriorAvatar: warriorProfile.avatar,
        }
        const validName = validateName(body.warriorName)

        let noFormErrors = true

        if (!validName.valid) {
            noFormErrors = false
            notifications.danger(validName.error, 1500)
        }

        if (noFormErrors) {
            xfetch(`/api/warrior/${$warrior.id}`, { body })
                .then(res => res.json())
                .then(function(updatedWarrior) {
                    warrior.update({
                        id: warriorProfile.id,
                        name: warriorProfile.name,
                        email: warriorProfile.email,
                        rank: warriorProfile.rank,
                        avatar: warriorProfile.avatar,
                    })

                    notifications.success('Profile updated.', 1500)
                    eventTag('update_profile', 'engagement', 'success')
                })
                .catch(function(error) {
                    notifications.danger(
                        $_('pages.warriorProfile.errorUpdating'),
                    )
                    eventTag('update_profile', 'engagement', 'failure')
                })
        }
    }

    function updateWarriorPassword(e) {
        e.preventDefault()
        const body = {
            warriorPassword1,
            warriorPassword2,
        }
        const validPasswords = validatePasswords(
            warriorPassword1,
            warriorPassword2,
        )

        let noFormErrors = true

        if (!validPasswords.valid) {
            noFormErrors = false
            notifications.danger(validPasswords.error, 1500)
        }

        if (noFormErrors) {
            xfetch('/api/auth/update-password', { body })
                .then(function() {
                    notifications.success(
                        $_('pages.warriorProfile.passwordUpdated'),
                        1500,
                    )
                    updatePassword = false
                    eventTag('update_password', 'engagement', 'success')
                })
                .catch(function(error) {
                    notifications.danger(
                        $_('pages.warriorProfile.passwordUpdateError'),
                    )
                    eventTag('update_password', 'engagement', 'failure')
                })
        }
    }

    onMount(() => {
        if (!$warrior.id) {
            router.route('/enlist')
        }
    })

    $: updateDisabled = warriorProfile.name === ''
    $: updatePasswordDisabled =
        warriorPassword1 === '' ||
        warriorPassword2 === '' ||
        appConfig.authMethod === 'ldap'
</script>

<PageLayout>
    <div class="flex justify-center">
        <div class="w-full md:w-1/2 lg:w-1/3">
            {#if !updatePassword}
                <form
                    on:submit="{updateWarriorProfile}"
                    class="bg-white shadow-lg rounded p-4 md:p-6 mb-4"
                    name="updateProfile">
                    <h2
                        class="font-bold text-xl md:text-2xl mb-2 md:mb-6
                        md:leading-tight">
                        {$_('pages.warriorProfile.title')}
                    </h2>

                    <div class="mb-4">
                        <label
                            class="block text-gray-700 text-sm font-bold mb-2"
                            for="yourName">
                            {$_('pages.warriorProfile.fields.name.label')}
                        </label>
                        <input
                            bind:value="{warriorProfile.name}"
                            placeholder="{$_('pages.warriorProfile.fields.name.placeholder')}"
                            class="bg-gray-200 border-gray-200 border-2
                            appearance-none rounded w-full py-2 px-3
                            text-gray-700 leading-tight focus:outline-none
                            focus:bg-white focus:border-purple-500"
                            id="yourName"
                            name="yourName"
                            type="text"
                            required />
                    </div>

                    <div class="mb-4">
                        <label
                            class="block text-gray-700 text-sm font-bold mb-2"
                            for="yourEmail">
                            {$_('pages.warriorProfile.fields.email.label')}
                            {#if warriorProfile.verified}
                                <span
                                    class="font-bold text-green-600
                                    border-green-500 border py-1 px-2 rounded
                                    ml-1">
                                    {$_('pages.warriorProfile.fields.email.verified')}
                                </span>
                            {/if}
                        </label>
                        <input
                            bind:value="{warriorProfile.email}"
                            class="bg-gray-200 border-gray-200 border-2
                            appearance-none rounded w-full py-2 px-3
                            text-gray-700 leading-tight focus:outline-none
                            cursor-not-allowed"
                            id="yourEmail"
                            name="yourEmail"
                            type="email"
                            disabled />
                    </div>

                    {#if avatarService == 'dicebear' || avatarService == 'gravatar' || avatarService == 'robohash'}
                        <div class="mb-4">
                            <label
                                class="block text-gray-700 text-sm font-bold
                                mb-2"
                                for="yourAvatar">
                                {$_('pages.warriorProfile.fields.avatar.label')}
                            </label>
                            <div class="flex">
                                <div class="md:w-2/3 lg:w-3/4">
                                    <div class="relative">
                                        <select
                                            bind:value="{warriorProfile.avatar}"
                                            class="block appearance-none w-full
                                            border-2 border-gray-400
                                            text-gray-700 py-3 px-4 pr-8 rounded
                                            leading-tight focus:outline-none
                                            focus:border-purple-500"
                                            id="yourAvatar"
                                            name="yourAvatar">
                                            {#each avatars as item}
                                                <option value="{item}">
                                                    {item}
                                                </option>
                                            {/each}
                                        </select>
                                        <div
                                            class="pointer-events-none absolute
                                            inset-y-0 right-0 flex items-center
                                            px-2 text-gray-700">
                                            <DownCarrotIcon />
                                        </div>
                                    </div>
                                </div>
                                <div class="md:w-1/3 lg:w-1/4 ml-1">
                                    <span class="float-right">
                                        <WarriorAvatar
                                            warriorId="{warriorProfile.id}"
                                            avatar="{warriorProfile.avatar}"
                                            {avatarService}
                                            width="40" />
                                    </span>
                                </div>
                            </div>
                        </div>
                    {/if}

                    <div>
                        <div class="text-right">
                            <button
                                type="button"
                                class="inline-block align-baseline font-bold
                                text-sm text-blue-500 hover:text-blue-800 mr-4"
                                on:click="{toggleUpdatePassword}">
                                {$_('pages.warriorProfile.updatePasswordButton')}
                            </button>
                            <SolidButton
                                type="submit"
                                disabled="{updateDisabled}">
                                {$_('pages.warriorProfile.saveProfileButton')}
                            </SolidButton>
                        </div>
                    </div>
                </form>
            {/if}

            {#if updatePassword}
                <form
                    on:submit="{updateWarriorPassword}"
                    class="bg-white shadow-lg rounded p-6 mb-4"
                    name="updateWarriorPassword">
                    <div
                        class="font-bold text-xl md:text-2xl mb-2 md:mb-6
                        md:leading-tight text-center">
                        {$_('pages.warriorProfile.updatePasswordForm.title')}
                    </div>

                    <div class="mb-4">
                        <label
                            class="block text-gray-700 text-sm font-bold mb-2"
                            for="yourPassword1">
                            {$_('pages.warriorProfile.updatePasswordForm.fields.password.label')}
                        </label>
                        <input
                            bind:value="{warriorPassword1}"
                            placeholder="{$_('pages.warriorProfile.updatePasswordForm.fields.password.placeholder')}"
                            class="bg-gray-200 border-gray-200 border-2
                            appearance-none rounded w-full py-2 px-3
                            text-gray-700 leading-tight focus:outline-none
                            focus:bg-white focus:border-purple-500"
                            id="yourPassword1"
                            name="yourPassword1"
                            type="password"
                            required />
                    </div>

                    <div class="mb-4">
                        <label
                            class="block text-gray-700 text-sm font-bold mb-2"
                            for="yourPassword2">
                            {$_('pages.warriorProfile.updatePasswordForm.fields.confirmPassword.label')}
                        </label>
                        <input
                            bind:value="{warriorPassword2}"
                            placeholder="{$_('pages.warriorProfile.updatePasswordForm.fields.confirmPassword.placeholder')}"
                            class="bg-gray-200 border-gray-200 border-2
                            appearance-none rounded w-full py-2 px-3
                            text-gray-700 leading-tight focus:outline-none
                            focus:bg-white focus:border-purple-500"
                            id="yourPassword2"
                            name="yourPassword2"
                            type="password"
                            required />
                    </div>

                    <div class="text-right">
                        <button
                            type="button"
                            class="inline-block align-baseline font-bold text-sm
                            text-blue-500 hover:text-blue-800 mr-4"
                            on:click="{toggleUpdatePassword}">
                            {$_('pages.warriorProfile.updatePasswordForm.cancelButton')}
                        </button>
                        <SolidButton
                            type="submit"
                            disabled="{updatePasswordDisabled}">
                            {$_('pages.warriorProfile.updatePasswordForm.saveButton')}
                        </SolidButton>
                    </div>
                </form>
            {/if}
        </div>
    </div>
</PageLayout>
