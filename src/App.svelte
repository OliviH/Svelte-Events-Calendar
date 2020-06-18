<script>
	import { onMount } from 'svelte'
	import { afterUpdate } from 'svelte'

	let today = new Date(),
		currentMonth = today.getMonth(),
		currentYear = today.getFullYear(),
		currentDay = today.getDate(),
		dayactive = (currentMonth === today.getMonth()&&currentYear === today.getFullYear()),
		firstDay = (new Date(currentYear, currentMonth)).getDay(),
		monthLength = new Date(today.getFullYear(), today.getMonth()+1, 0).getDate(),
		months = [
			"Janvier",
			"Février",
			"Mars",
			"Avril",
			"Mai",
			"Juin",
			"Juillet",
			"Août",
			"Septembre",
			"Octobre",
			"Novembre",
			"Décembre"
		],
		days = Array.from(Array(monthLength), (_, i) => i + 1),
		events = null,
		arrDisplayEvents = [],
		dateSelected,
		indexEventDisplay,
		numberEventsDisplayed = 3,
		nextEvents = false,
		previousEvents = false

$:{
	//document.querySelector(["#iddateselector"]).valueAsDate = new Date(dateSelected)
	//console.log("dateSelected",dateSelected)
//	if(document.querySelector('#iddateselector'))document.querySelector('#iddateselector').valueAsDate = new Date()
	selectDisplay(dateSelected)
}



	onMount(()=>{
		dateSelected = dateForInputDate(currentYear,currentMonth,currentDay)
		fetch('http://localhost:3003/?nb='+(getRandomInt(50)+1))
		.then(resp=>{return resp.json()})
		.then(constructCalendar)
		.catch(e=>console.warn(e))
	})

	const handleDatechange = (e) => {
		let datechange = new Date(e.target.value)
		selectorDate(
			datechange.getFullYear(),
			datechange.getMonth()
		)
		selectDisplay(
			datechange
		)
	}

	const dateForInputDate = (day, month, year) => {
		return [
				year,
				(month+1).toString().padStart(2, '0'),
				day.toString().padStart(2, '0')
			].join("-")
	}

	afterUpdate(()=> {
		addBadges()
	})

	const selectorDate = (fullyear, month) => {
		currentMonth = month
		currentYear = fullyear
		monthLength = new Date(currentYear, currentMonth+1, 0).getDate()
		days = Array.from(Array(monthLength), (_, i) => i + 1)
		firstDay = (new Date(currentYear, currentMonth)).getDay()
		dayactive = (currentMonth === today.getMonth()&&currentYear === today.getFullYear())
	}

	const next = () => {
		selectorDate(
			(currentMonth === 11) ? currentYear + 1 : currentYear,
			(currentMonth + 1) % 12
		)
	}
	const previous = () => {
		selectorDate(
			(currentMonth === 0) ? currentYear - 1 : currentYear,
			(currentMonth === 0) ? 11 : currentMonth - 1
		)
	}

	const datenow = () => {
		selectorDate(
			today.getFullYear(),
			today.getMonth()
		)
	}

	const handleDateClick = (e) => {
		//console.log(e.target.dataset, `${e.target.dataset.year}-${e.target.dataset.month+1}-${e.target.dataset.day}`)
		selectDisplay(new Date(`${e.target.dataset.year}-${parseInt(e.target.dataset.month)+1}-${e.target.dataset.day}`))
	}

	const constructCalendar = (datas) => {
		const listevents = []
		datas.forEach(dtelem=>{
			if(dtelem.events)dtelem.events.forEach(ev=>{
					if(ev.date){
						let nedate = new Date(ev.date)
						ev.id_ref = `id_cal_day_${nedate.getFullYear()}_${nedate.getMonth()}_${nedate.getDate()}`
					}
					ev.name = dtelem.name
					ev.phone = dtelem.phone
					ev.email = dtelem.email
					listevents.push(ev)
				})
		})
		events = listevents.slice().sort((a, b) => new Date(b.date) - new Date(a.date)).reverse()
		//console.log("events", events)
		selectDisplay(today)
		addBadges()
	}

	const selectDisplay = (date) => {
		if(events){
			indexEventDisplay = indexFromStartDate(events, date)
			//console.log("indexEventDisplay",indexEventDisplay)
			if(indexEventDisplay!==false) {
				getDisplayEventsFromIndex(indexEventDisplay)
			}

		}
	}

	const getDisplayEventsFromIndex = (index) => {
		if(events){
			if(index!==false) {
				arrDisplayEvents = _nbElementsArray(events, numberEventsDisplayed, index)
			} else {
				arrDisplayEvents= []
			}
			nextEvents = ((indexEventDisplay+numberEventsDisplayed)<events.length)?(events.length-(indexEventDisplay+numberEventsDisplayed)): false
			let previous = events.length - (events.length - indexEventDisplay)
			previousEvents = (previous>0)? previous: false
		}
	}

	const handleBtnClickEventPrevious = () => {
		if(indexEventDisplay!==false) {
			console.log("indexEventDisplay",indexEventDisplay,numberEventsDisplayed)
			indexEventDisplay = (indexEventDisplay-numberEventsDisplayed>0)
				?indexEventDisplay-numberEventsDisplayed
				:0
			console.log("indexEventDisplay",indexEventDisplay)
			getDisplayEventsFromIndex(indexEventDisplay)
		}
	}

	const handleBtnClickEventNext = () => {
		if(indexEventDisplay!==false) {
			indexEventDisplay = indexEventDisplay+numberEventsDisplayed
			getDisplayEventsFromIndex(indexEventDisplay)
		}
	}


	const indexFromStartDate = (arr, date) => {
		let index = false,
			indexintermediate = 1
		if(arr) {
			for (let i = 0; i < arr.length; i++){
				if(new Date(arr[i].date).getTime() >= date.getTime()) {
					index = i
					break
				}
				index = i+1
			}
			return index
		} else {
			return false
		}
	}

	const addBadges = () => {
		const matches  = document.querySelectorAll(`.badge`)
		matches.forEach(item=>item.remove())
		if(events)events.forEach(ev=>{
			if(document.querySelector(`#${ev.id_ref}`)) {
				if(document.querySelector(`#${ev.id_ref} > .badge`)) {
					let valExists = parseInt(document.querySelector(`#${ev.id_ref} > .badge`).innerText)
					document.querySelector(`#${ev.id_ref} > .badge`).innerText = valExists++
				} else {
					document.querySelector(`#${ev.id_ref}`).insertAdjacentHTML(
						`beforeend`,
						`<div class="badge" data-ref="${ev.id_ref}">1</div>`
					)
				}
			}
		})
	}

	const _nbElementsArray = (arr, numberofelements, fromIndex = 0) => {
		return (arr)?arr.slice(fromIndex, fromIndex+numberofelements):[]
	}

	const getRandomInt = (max) => { return Math.floor(Math.random() * Math.floor(max)) }
</script>
<style>
	.container {
		width: 100%;
		height: 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
		font-size:.9em;
	}
	.calendar-card {
		background-color: #fff;
		margin: 1em auto 0 auto;
		padding: 1.2em;
		border: 2px solid var(--blue-grey-200);
		border-radius: 8px;
		/* box-shadow: 0 8px 6px -6px rgba(0,0,0,.5); */
		user-select: none;
		width: 370px;
		min-height: 350px;
	}
	.grid-month {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: center;
	}
	.month {
		color: var(--blue-grey-700);
		font-weight: 500;
		display: flex;
		align-items: center;
	}
	svg {
		fill: var(--blue-grey-200);
	}
	.date-grid {
		display: grid;
		grid-template-columns: repeat(7, 1fr);
	}
	.day {
		font-size: 0.7em;
		color: var(--blue-grey-400);
		font-weight: 500;
		letter-spacing: 0.1em;
		font-variant: small-caps;
		text-align: center;
	}
	.day-ref {
		margin-top: 1.25em;
	}
	.day-grid {
		margin-top: .5em;
	}
	.day-grid button {
		position: relative;
		border: 0;
		width: 4.5ch;
		height: 4.5ch;
		border-radius: 50%;
		background-color: transparent;
		color: var(--blue-grey-600);
	}
	.pos1 {
		grid-column: 1
	}
	.pos2 {
		grid-column: 2
	}
	.pos3 {
		grid-column: 3
	}
	.pos4 {
		grid-column: 4
	}
	.pos5 {
		grid-column: 5
	}
	.pos6 {
		grid-column: 6
	}
	.pos7 {
		grid-column: 7
	}
	.btn {
		cursor: pointer;
	}
	.btn * {
		pointer-events: none;
	}
	.today {
		margin-right: 1.25em;
	}
	button:hover {
		outline: none;
		background-color: var(--blue-grey-050);
		color: var(--blue-grey-700);
	}
	button .active {
		outline: none;
		background-color: var(--orange-brown-200);
		color: var(--blue-grey-600);
	}
	.events-pan {
		background-color: #fff;
		margin: 1em auto 0 auto;
		padding: 1.2em;
		border: 2px solid var(--blue-grey-200);
		border-radius: 8px;
		width: 370px;
		max-width: calc( );
	}
	.events-title {
		color: var(--blue-grey-700);
		font-weight: 500;
		display: flex;
		align-items: center;
	}
	.events-title svg {
		margin-right:1em;
	}
	.card{
		display: grid;
		margin: .3em 0 .7em 0;
		padding: .2em;
		border: thin solid var(--blue-grey-200);
		border-radius: 5px;
		grid-template-columns: repeat(2, 50%);
		font-size: .6em;
	}
	.title{
		grid-column: 1 / span 2;
		font-weight: 500;
		color: var(--teal-700);
		text-align: justify;
		padding: .3em;
	}
	.date{
		grid-column: 1 / span 2;
		font-weight: 500;
		color: var(--red-dark-300);
		text-align: right;
		padding: .3em;
	}
	.name{
		border-top: thin dashed var(--teal-200);
		color: var(--blue-grey-300);
		text-align: center;
		grid-column: 1 / span 2;
		padding-top:.3em;
	}
	.phone{
		color: var(--blue-grey-200);
		text-align: left;
		padding: .3em;
	}
	.email{
		color: var(--blue-grey-200);
		text-align: right;
		padding: .3em;
	}
	.events-searchdate {
		font-size: .5em;
		color: var(--blue-grey-300);
		display: flex;
		justify-content: center;
	}
	.events-searchdate input {
		margin: .3em auto;
		width: 90%;
		outline: none;
		border: thin solid var(--blue-grey-100);
	}
	.day-tag {
		position: relative;
	}
</style>
<div class="container">
	<div class="calendar-card">
		<div class="grid-month">
			<div class="btn" on:click={previous} >
				<svg width="24" height="24" viewBox="0 -65.5 528 480"><path d="M256 400c114.686 0 208 -93.3145 208 -208s-93.3145 -208 -208 -208s-208 93.3145 -208 208s93.3145 208 208 208zM256 368c-97.3916 0 -176 -78.6084 -176 -176s78.6084 -176 176 -176s176 78.6084 176 176s-78.6084 176 -176 176zM244.5 294.5l23 -22.5l-64 -64h164.5 v-32h-164.5l64 -64l-23 -22.5l-91 91l-11.5 11.5l11.5 11.5z"></path></svg>
			</div>
			<div class="month">
				<div class="btn today" on:click={datenow}>
					<svg width="24" height="24" viewBox="0 -65.5 528 480"><path d="M256 384c105.85 0 192 -86.1504 192 -192s-86.1504 -192 -192 -192s-192 86.1504 -192 192s86.1504 192 192 192zM256 352c-88.5547 0 -160 -71.4453 -160 -160s71.4453 -160 160 -160s160 71.4453 160 160s-71.4453 160 -160 160zM256 240c26.5098 0 48 -21.4902 48 -48 s-21.4902 -48 -48 -48s-48 21.4902 -48 48s21.4902 48 48 48z"></path></svg>
				</div>
				{months[currentMonth]} {currentYear}
			</div>
			<div class="btn" on:click={next} >
				<svg width="24" height="24" viewBox="0 -65.5 528 480"><path d="M256 400c114.686 0 208 -93.3145 208 -208s-93.3145 -208 -208 -208s-208 93.3145 -208 208s93.3145 208 208 208zM256 368c-97.3916 0 -176 -78.6084 -176 -176s78.6084 -176 176 -176s176 78.6084 176 176s-78.6084 176 -176 176zM267.5 294.5l91 -91l11.5 -11.5 l-11.5 -11.5l-91 -91l-23 22.5l64 64h-164.5v32h164.5l-64 64z"></path></svg>
			</div>
		</div>
		<div class="date-grid day-ref">
			<div class="day">Lu</div>
			<div class="day">Ma</div>
			<div class="day">Me</div>
			<div class="day">Je</div>
			<div class="day">Ve</div>
			<div class="day">Sa</div>
			<div class="day">Di</div>
		</div>
		<div class="date-grid day-grid">
			{#each days as day}
				<div class="day-tag" id={`id_cal_day_${currentYear}_${currentMonth}_${day}`}>
					<button 
						class={
							(day===1&&day===currentDay&&dayactive)?`pos${firstDay} active`:(day===1)?`pos${firstDay}`:(day===currentDay&&dayactive)?"active":null
						}
						on:click={handleDateClick}
						data-day={day}
						data-month={currentMonth}
						data-year={currentYear}
					>{day}</button>
					{#if day.badge}
						<div class={(day.badgeType)?`badge badge-${day.badgeType}`:`badge`}>{day.badge}</div>
					{/if}
				</div>
			{/each}
		</div>
	</div>
	{#if events}
	<div class="events-pan">
		<div class="events-title">
			<svg width="24" height="24" viewBox="0 -65.5 528 480" ><path d="M64 368h16h64h16v-16v-64v-16h-16h-64h-16v16v64v16zM96 336v-32h32v32h-32zM192 336h240v-32h-240v32zM64 240h16h64h16v-16v-64v-16h-16h-64h-16v16v64v16zM96 208v-32h32v32h-32zM192 208h240v-32h-240v32zM64 112h16h64h16v-16v-64v-16h-16h-64h-16v16v64v16zM96 80 v-32h32v32h-32zM192 80h240v-32h-240v32z"></path></svg>
			<div>{events.length} événement{(events.length>1)?"s":""} référencé{(events.length>1)?"s":""}</div>
		</div>
		<div class="events-searchdate">
			{#if previousEvents}
			<div 
				class="btn"
				on:click={handleBtnClickEventPrevious}
			>
				<svg width="24" height="24" viewBox="0 -65.5 528 480"><path d="M256 400c114.686 0 208 -93.3145 208 -208s-93.3145 -208 -208 -208s-208 93.3145 -208 208s93.3145 208 208 208zM256 368c-97.3916 0 -176 -78.6084 -176 -176s78.6084 -176 176 -176s176 78.6084 176 176s-78.6084 176 -176 176zM244.5 294.5l23 -22.5l-64 -64h164.5 v-32h-164.5l64 -64l-23 -22.5l-91 91l-11.5 11.5l11.5 11.5z"></path></svg>
			</div>
			{/if}
			<input type="date" on:change={handleDatechange} id="iddateselector">
			{#if nextEvents}
			<div 
				class="btn"
				on:click={handleBtnClickEventNext}
			>
				<svg width="24" height="24" viewBox="0 -65.5 528 480"><path d="M256 400c114.686 0 208 -93.3145 208 -208s-93.3145 -208 -208 -208s-208 93.3145 -208 208s93.3145 208 208 208zM256 368c-97.3916 0 -176 -78.6084 -176 -176s78.6084 -176 176 -176s176 78.6084 176 176s-78.6084 176 -176 176zM267.5 294.5l91 -91l11.5 -11.5 l-11.5 -11.5l-91 -91l-23 22.5l64 64h-164.5v32h164.5l-64 64z"></path></svg>
			</div>
			{/if}
		</div>
		<div class="events-content">

			{#each arrDisplayEvents as event}
				<div class="card">
					<div class="title">{event.evenement}</div>
					<div class="date">{new Date(event.date).toLocaleDateString()}</div>
					<div class="name">{event.name}</div>
					<div class="phone">{event.phone}</div>
					<div class="email">{event.email}</div>
				</div>
			{/each}
		</div>
	</div>
	{/if}
</div>