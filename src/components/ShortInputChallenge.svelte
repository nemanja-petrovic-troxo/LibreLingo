<script>
    import { slide } from "svelte/transition"
    import { onMount } from "svelte"
    import hotkeys from "hotkeys-js"
    import levenshtein from "js-levenshtein"
    import shuffle from "lodash.shuffle"
    import ChallengePanel from "./ChallengePanel"

    export let challenge
    export let registerResult
    export let resolveChallenge
    export let languageName
    export let languageCode
    let answer = null
    let submitted = false
    let correct = null
    let spellingSuggestion = ""
    let inputFieldRef = null
    let picture = shuffle(challenge.pictures)[0]

    $: submitChallenge = () => {
        if (!answer) return
        if (submitted) return
        correct = false

        challenge.formInTargetLanguage.forEach(form => {
            if (
                levenshtein(
                    answer
                        .toLowerCase()
                        .replace(/^\s+|\s+$/g, "")
                        .replace(/\s+/g, " "),
                    form.toLowerCase()
                ) <= 1
            ) {
                correct = true
                spellingSuggestion =
                    form
                        .replace(/^\s+|\s+$/g, "")
                        .replace(/\s+/g, " ")
                        .toLowerCase() === answer.toLowerCase()
                        ? ""
                        : `You made a small error. Correct spelling: ${form}`
            }
        })

        registerResult(correct)
        inputFieldRef.blur()
        submitted = true
    }

    $: finishChallenge = () => {
        answer = null
        submitted = false
        resolveChallenge()
    }

    const focusMe = el => {
        setTimeout(() => {
            if (el.disabled) {
                el.blur()
            } else {
                el.focus()
            }
        }, 1)
    }

    onMount(() => {
        hotkeys.unbind("enter")
        hotkeys("enter", () => {
            if (submitted) {
                finishChallenge()
            } else {
                submitChallenge()
            }
        })
    })
</script>

<form on:submit|preventDefault="{submitChallenge}">
    <div class="section">
        <p
            class="is-size-1 is-size-2-tablet is-size-4-mobile has-text-centered">
            Type
            <b>{challenge.meaningInSourceLanguage}</b>
            in {languageName}!
        </p>
    </div>
    <div class="columns">

        <div class="column">
            <input
                tabindex="0"
                data-test="answer"
                type="text"
                class="input"
                autofocus
                placeholder="Type your answer…"
                disabled="{submitted}"
                spellcheck="false"
                lang="{languageCode}"
                use:focusMe
                bind:this="{inputFieldRef}"
                bind:value="{answer}" />
        </div>
        <div class="column is-one-fourth">
            <div class="card">
                <div class="card-image">
                    <figure class="image is-1by1">
                        <img src="images/{picture}" alt="" />
                    </figure>
                </div>
            </div>
        </div>
    </div>

    {#if answer && !submitted}
        <ChallengePanel message="" buttonText="Submit" submit />
    {/if}

    {#if submitted}
        {#if !correct}
            <ChallengePanel
                message="Incorrect solution!"
                messageDetail="{`Correct answer: ${challenge.formInTargetLanguage[0]}`}"
                buttonText="Continue"
                incorrect
                buttonAction="{finishChallenge}" />
        {/if}
        {#if correct}
            <ChallengePanel
                message="Correct solution!"
                messageDetail="{spellingSuggestion}"
                buttonText="Continue"
                correct
                buttonAction="{finishChallenge}" />
        {/if}
    {/if}

</form>

<style>
    .card {
        max-width: 16em;
        margin: auto;
    }
</style>
