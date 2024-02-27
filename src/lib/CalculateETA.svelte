<script lang="ts">
  import { Holiday } from 'open-holiday-js'
  import { onMount } from 'svelte'

  let selectedData: any[] = []
  let selectedSubdivision: string

  let startDate: Date = new Date()
  let endDate: Date = new Date()

  let transferDate: string = ''
  let expectedArrival: string = ''

  let arrivalDates: Date[] = []
  let possibleArrivalDates: Date[] = []
  let holidays: Date[] = []

  onMount(async () => {
    await subdivisions()
  })

  const subdivisions = async () => {
    try {
      const response = await holiday.getSubdivisions('DE', 'DE')
      if (!response) throw Error
      selectedData = response.map(({ code, name }) => ({
        code,
        name: name[0].text,
      }))
    } catch (error) {
      console.log(error)
    }
  }

  const handleDateChange = (event: Event) => {
    const newStartDate = (event.target as HTMLInputElement)?.value
    if (newStartDate !== undefined) {
      startDate = new Date(newStartDate)
      const newEndDate = new Date(
        startDate.getFullYear(),
        startDate.getMonth(),
        startDate.getDate() + 7
      )
      endDate = new Date(newEndDate)
    }
    transferDate = startDate.toLocaleDateString('de-DE', {
      weekday: 'long',
      month: 'long',
      day: 'numeric',
      year: 'numeric',
    })
  }

  const holiday = new Holiday()
  const getHolidays = async () => {
    const response = await holiday.getPublicHolidays(
      'DE',
      startDate,
      endDate,
      selectedSubdivision
    )

    for (let index = 0; index < response.length; index++) {
      const element = response[index]
      const startDate = element.startDate
      if (startDate.getDay() > 0 && startDate.getDay() < 6)
        holidays.push(startDate)
    }

    for (let index = 0; index < 7; index++) {
      const date = new Date(
        startDate.getFullYear(),
        startDate.getMonth(),
        startDate.getDate() + index
      )
      if (date.getDay() > 0 && date.getDay() < 6) arrivalDates.push(date)
    }
  }

  const calculateETA = async () => {
    await getHolidays()

    for (let i = 0; i < arrivalDates.length; i++) {
      let isHoliday = false
      for (let j = 0; j < holidays.length; j++) {
        if (arrivalDates[i].getTime() === holidays[j].getTime()) {
          isHoliday = true
          break
        }
      }
      if (!isHoliday) {
        possibleArrivalDates.push(arrivalDates[i])
      }
    }

    if (possibleArrivalDates[0].getDate() == startDate.getDate())
      possibleArrivalDates.splice(0, 1)

    if (startDate.getHours() < 15)
      expectedArrival = possibleArrivalDates[0].toLocaleDateString('de-DE', {
        weekday: 'long',
        month: 'long',
        day: 'numeric',
        year: 'numeric',
      })
    else
      expectedArrival = possibleArrivalDates[1].toLocaleDateString('de-DE', {
        weekday: 'long',
        month: 'long',
        day: 'numeric',
        year: 'numeric',
      })
  }
</script>

<div>
  <select bind:value={selectedSubdivision}>
    <option value="" disabled selected>Select a State...</option>
    {#each selectedData as sub, index}
      <option value={sub.code}>{sub.name}</option>
    {/each}
  </select>
  <input
    type="datetime-local"
    name="startdate"
    id="startdate"
    bind:value={startDate}
    on:change={handleDateChange}
  />
  <button on:click={calculateETA}> Calculate </button>
  <p>
    Day and time of the transfer: {transferDate}
  </p>
  <p>
    Expected arrival: {expectedArrival}
  </p>
</div>
