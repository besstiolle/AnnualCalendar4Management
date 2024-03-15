<script lang="ts">
    import { browser } from '$app/environment';
    import { google, outlook, office365, yahoo, ics } from "calendar-link";

    let entryDateForm:HTMLInputElement
    let emailHimForm:HTMLInputElement
    let entryDate:Date
    let emailHim:string
    let error:string=""

    //Liste des évènements de calendrier
    let allEvents:CalendarEvent[]=[]

    class CalendarEvent{
        type:TypeEvent
        date:Date
        anciennete:number=0

        constructor(type:TypeEvent, date:Date, anciennete:number=0){
            this.type = type
            this.date = date
            this.anciennete = anciennete
        }
    }

    enum TypeEvent{
        TODAY,
        ANNIVERSAIRE,
        EA, //Entretien Annuel
        EP, //Entretien Professionnel, une fois toutes les 2 ans
        ST, //Suivit Technique, anciennement Suivi DT
        OOO, // One on One
        CP, //Career Path
    }

    function update(){

        allEvents = []
        error = ""
        
        let goodFormat = entryDateForm.value.substring(6,10) + "-" + entryDateForm.value.substring(3,5) + "-" + entryDateForm.value.substring(0,2)
        let entryDate = new Date(goodFormat)
        //Test rapide sur format de date
        if(entryDateForm.value.length!= 10 || !(entryDate instanceof Date) || isNaN(entryDate.getDate())) {
            error = "Format de date non supporté."
            return
        } 

        //TODO : Gérer cas de date > date du jour

        /*let emailHim = emailHimForm.value.trim()
        if(emailHim == ""){
            error = "Email non renseigné"
            return
        }*/

        // Date du jour rapporté à 0h00:00
        let today = new Date(new Date().setHours(0, 0, 0, 0))

        //2 cas de figure. La date d'entrée reportée sur l'année de TODAY est : 
        //  Située AVANT TODAY : On va générer l'anniversaire sur l'année N et N+1
        //  Située APRES TODAY : **************************************** N et N-1
        let entryDateCurrentYear = new Date(entryDate)
        entryDateCurrentYear.setFullYear(today.getFullYear())

        let differentiel = 1
        if(entryDateCurrentYear > today){
            differentiel = -1
        }
        let entryDateOtherYear = new Date(entryDateCurrentYear)
        entryDateOtherYear.setFullYear(entryDateOtherYear.getFullYear() + differentiel)

        allEvents.push(new CalendarEvent(TypeEvent.TODAY, today))

        //Ajouter tous les évènements déroulés sur l'année en cours puis idem avec l'autre année (N+1 ou N-1)
        allEvents = allEvents.concat(generateOneYear(entryDateCurrentYear, entryDateCurrentYear.getFullYear() - entryDate.getFullYear()))
        allEvents = allEvents.concat(generateOneYear(entryDateOtherYear, entryDateOtherYear.getFullYear() - entryDate.getFullYear()))

        //Trier les évènements
        allEvents.sort(compareCalendarEvent)
    }

    function generateOneYear(date:Date, diff:number):CalendarEvent[]{
        let eventsOfYear:CalendarEvent[] = []
        eventsOfYear.push(new CalendarEvent(TypeEvent.ANNIVERSAIRE, date, diff))
        if(diff % 2 == 0) {
            eventsOfYear.push(new CalendarEvent(TypeEvent.EP, date))
        } else {
            eventsOfYear.push(new CalendarEvent(TypeEvent.EA, date))
        }
        
        //Suivi technique
        let dateST = new Date(date)
        dateST.setMonth(dateST.getMonth() + 6)
        eventsOfYear.push(new CalendarEvent(TypeEvent.ST, dateST))
        
        //One on One M+3  M+9
        let dateOOO3 = new Date(date)
        dateOOO3.setMonth(dateOOO3.getMonth() + 3)
        eventsOfYear.push(new CalendarEvent(TypeEvent.OOO, dateOOO3))
        let dateOOO9 = new Date(date)
        dateOOO9.setMonth(dateOOO9.getMonth() + 9)
        eventsOfYear.push(new CalendarEvent(TypeEvent.OOO, dateOOO9))

        //Career Path
        let dateCP = new Date(date)
        dateCP.setMonth(dateCP.getMonth() + 10)
        eventsOfYear.push(new CalendarEvent(TypeEvent.CP, dateCP))

        return eventsOfYear
    }

    function compareCalendarEvent(ob1:CalendarEvent, ob2:CalendarEvent):number{
        if(ob1.date > ob2.date){
            return 1
        } else if(ob1.date < ob2.date) {
            return -1
        } else {
            return 0
        }
    }

    function toTitle(type:number):string{
        switch(type){
            case TypeEvent.ANNIVERSAIRE : return "Date d'anniversaire";
            case TypeEvent.EA : return "Entretien Annuel";
            case TypeEvent.EP : return "Entretien Professionnel";
            case TypeEvent.ST : return "Suivi Technique";
            case TypeEvent.OOO : return "One On One";
            case TypeEvent.CP : return "Career Path"
            case TypeEvent.TODAY : return "Aujourd'hui";
            default: return "Evenement inconnu"
        } 
    }
</script>

<h1>Welcome to the app</h1>
<p>A priori vous savez pourquoi vous êtes là.</p>

<label for="entryDate">Date d'arrivée dans l'entreprise (format DD/MM/YYYY)</label>
<input id="entryDate" type="text" bind:this={entryDateForm} value="15/09/2001"/>
<label for="emailHim">L'email du copain</label>
<input id="emailHim" type="text" bind:this={emailHimForm}/>
<input type="button" title="go" value="go" on:click={update} />
{#if error != ""}<div class='error'>{error}</div>{/if}

{#if allEvents.length > 0}
    <h2>Evènements:</h2>
    {#each allEvents as evenement}
        <h3>{toTitle(evenement.type)}{#if evenement.type == TypeEvent.ANNIVERSAIRE} - {evenement.anciennete} an(s) d'exp{/if}</h3>
        <p>Date : {evenement.date.getDate()}/{evenement.date.getMonth()+1}/{evenement.date.getFullYear()}</p>
    {/each}
{/if}