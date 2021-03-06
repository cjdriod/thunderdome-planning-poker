<script>
    import ExternalLinkIcon from './icons/ExternalLinkIcon.svelte'
    import AddPlan from './AddPlan.svelte'
    import HollowButton from './HollowButton.svelte'
    import ViewPlan from './ViewPlan.svelte'
    import JiraImport from './JiraImport.svelte'
    import { _ } from '../i18n'

    export let plans = []
    export let isLeader = false
    export let sendSocketEvent = () => {}
    export let eventTag
    export let notifications

    const defaultPlan = {
        id: '',
        name: '',
        type: $_('actions.plan.types.story'),
        referenceId: '',
        link: '',
        description: '',
        acceptanceCriteria: '',
    }

    let showAddPlan = false
    let showViewPlan = false
    let selectedPlan = { ...defaultPlan }
    let showCompleted = false

    const toggleAddPlan = planId => () => {
        if (planId) {
            selectedPlan = plans.find(p => p.id === planId)

            eventTag('plan_show_edit', 'battle', ``)
        } else {
            selectedPlan = { ...defaultPlan }

            eventTag('plan_show_add', 'battle', ``)
        }
        showAddPlan = !showAddPlan
    }

    const togglePlanView = planId => () => {
        if (planId) {
            selectedPlan = plans.find(p => p.id === planId)
            eventTag('plan_show_view', 'battle', ``)
        } else {
            selectedPlan = { ...defaultPlan }

            eventTag('plan_unshow_view', 'battle', ``)
        }
        showViewPlan = !showViewPlan
    }

    const handlePlanAdd = newPlan => {
        sendSocketEvent('add_plan', JSON.stringify(newPlan))
        eventTag('plan_add', 'battle', '')
    }

    const activatePlan = id => () => {
        sendSocketEvent('activate_plan', id)
        eventTag('plan_activate', 'battle', '')
    }

    const handlePlanRevision = updatedPlan => {
        sendSocketEvent('revise_plan', JSON.stringify(updatedPlan))
        eventTag('plan_revise', 'battle', '')
    }

    const handlePlanDeletion = planId => () => {
        sendSocketEvent('burn_plan', planId)
        eventTag('plan_burn', 'battle', '')
    }

    const toggleShowCompleted = show => () => {
        showCompleted = show
        eventTag('plans_show', 'battle', `completed: ${show}`)
    }

    $: pointedPlans = plans.filter(p => p.points !== '')
    $: unpointedPlans = plans.filter(p => p.points === '')

    $: plansToShow = showCompleted ? pointedPlans : unpointedPlans
</script>

<div class="bg-white shadow-lg mb-4 rounded">
    <div class="flex items-center bg-gray-200 p-4 rounded-t">
        <div class="w-1/2">
            <h3 class="text-2xl leading-tight font-bold">Plans</h3>
        </div>
        <div class="w-1/2 text-right">
            {#if isLeader}
                <JiraImport {handlePlanAdd} {notifications} {eventTag} />
                <HollowButton onClick="{toggleAddPlan()}">
                    {$_('actions.plan.add')}
                </HollowButton>
            {/if}
        </div>
    </div>

    <ul class="flex border-b border-gray-400">
        <li class="-mb-px {showCompleted ? '' : 'mr-1'}">
            <button
                class="{showCompleted ? 'hover:text-blue-600 text-blue-400' : 'border-b border-blue-500 text-blue-600 hover:text-blue-800'}
                bg-white inline-block py-4 px-4 font-semibold"
                on:click="{toggleShowCompleted(false)}">
                {$_('actions.plan.unpointed', {
                    values: { count: unpointedPlans.length },
                })}
            </button>
        </li>
        <li class="mr-1 {showCompleted ? 'mr-1' : ''}">
            <button
                class="{showCompleted ? 'border-b border-blue-500 text-blue-600 hover:text-blue-800' : 'hover:text-blue-600 text-blue-400'}
                bg-white inline-block py-4 px-4 font-semibold"
                on:click="{toggleShowCompleted(true)}">
                {$_('actions.plan.pointed', {
                    values: { count: pointedPlans.length },
                })}
            </button>
        </li>
    </ul>

    {#each plansToShow as plan (plan.id)}
        <div
            class="flex flex-wrap items-center border-b border-gray-400 p-4"
            data-testId="battlePlan"
            data-planName="{plan.name}">
            <div class="w-full lg:w-2/3 mb-4 lg:mb-0">
                <div
                    class="inline-block font-bold align-middle"
                    data-testId="battlePlanName">
                    {#if plan.link !== ''}
                        <a
                            href="{plan.link}"
                            target="_blank"
                            class="text-blue-800">
                            <ExternalLinkIcon />
                        </a>
                        &nbsp;
                    {/if}
                    <div
                        class="inline-block text-sm text-gray-500
                        border-gray-400 border px-1 rounded"
                        data-testId="battlePlanType">
                        {plan.type}
                    </div>
                    &nbsp;
                    {#if plan.referenceId}[{plan.referenceId}]&nbsp;{/if}
                    {plan.name}
                </div>
                &nbsp;
                {#if plan.points !== ''}
                    <div
                        class="inline-block font-bold text-green-600
                        border-green-500 border px-2 py-1 rounded ml-2"
                        data-testId="battlePlanPoints">
                        {plan.points}
                    </div>
                {/if}
            </div>
            <div class="w-full lg:w-1/3 text-right">
                <HollowButton color="blue" onClick="{togglePlanView(plan.id)}">
                    {$_('actions.plan.view')}
                </HollowButton>
                {#if isLeader}
                    {#if !plan.active}
                        <HollowButton
                            color="red"
                            onClick="{handlePlanDeletion(plan.id)}">
                            {$_('actions.plan.delete')}
                        </HollowButton>
                    {/if}
                    <HollowButton
                        color="purple"
                        onClick="{toggleAddPlan(plan.id)}">
                        {$_('actions.plan.edit')}
                    </HollowButton>
                    {#if !plan.active}
                        <HollowButton onClick="{activatePlan(plan.id)}">
                            {$_('actions.plan.activate')}
                        </HollowButton>
                    {/if}
                {/if}
            </div>
        </div>
    {/each}
</div>

{#if showAddPlan}
    <AddPlan
        {handlePlanAdd}
        toggleAddPlan="{toggleAddPlan()}"
        {handlePlanRevision}
        planId="{selectedPlan.id}"
        planName="{selectedPlan.name}"
        planType="{selectedPlan.type}"
        referenceId="{selectedPlan.referenceId}"
        planLink="{selectedPlan.link}"
        description="{selectedPlan.description}"
        acceptanceCriteria="{selectedPlan.acceptanceCriteria}"
        {notifications}
        {eventTag} />
{/if}

{#if showViewPlan}
    <ViewPlan
        togglePlanView="{togglePlanView()}"
        planName="{selectedPlan.name}"
        planType="{selectedPlan.type}"
        referenceId="{selectedPlan.referenceId}"
        planLink="{selectedPlan.link}"
        description="{selectedPlan.description}"
        acceptanceCriteria="{selectedPlan.acceptanceCriteria}" />
{/if}
