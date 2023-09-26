<script lang="ts">
  import MultiSelect from 'svelte-multiselect'
  import { onMount } from 'svelte';
  
  let announcementId = 1000

  const URL = 'http://127.0.0.1:8000'
  let announcementText = '';
  let selected: { label: string; value: string; }[] = []; 
  let recipients: {label: string, value: string}[] = []
  let announcementDateTime: string|null = '';
  let announcements: any[] = []
  
  type Employee = {
    employee_id: string
    contact_no: string
  }
   

  const getEmployees = async () => {
    const response = await fetch(URL + '/api/employees');
    const data = await response.json();
    return data;
  }

  const createEmployeeView = (employees : Employee[]) => {
    return employees.map(employee =>{
      return{
        label: employee.employee_id,
        value: employee.contact_no
      }
    })
  }

  const convertEmployeeViewToEmployee = (employees:{label: string, value: string }[]) =>{
    return employees.map(employee =>{
      return{
        employee_id: employee.label,
        contact_no: employee.value
      }
    })
  }

  const getAnnouncements = async () =>{
    const response = await fetch(URL + '/api/announcements');
    const data = await response.json();
    return data;
  }

  onMount(async () => {
    const employees = await getEmployees();
    recipients = createEmployeeView(employees);
    const announcements = await getAnnouncements();
  })
  
  const sendAnnouncement = async () => {
    let immediate = false
    if(announcementDateTime == ""){
      immediate = true
      announcementDateTime = null
    }

    const options = {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        announcement_id: "announcment_"+(announcementId +=1),
        message : announcementText,
        recipients: convertEmployeeViewToEmployee(selected),
        scheduled_time: announcementDateTime,
        immediate: immediate
      })
    }
    const response = await fetch(URL + '/api/announcements', options)
    const data = await response.json()

    console.log('Announcement text:', announcementText);
    console.log('Selected recipients:', selected);
    announcementText = ''; 
    selected = []; 
    announcementDateTime = '';
    announcements = await getAnnouncements()
  };
</script>

<form>
  <label for="announcementDateTime">Date and Time:</label>
  <input type="datetime-local" id="announcementDateTime" bind:value={announcementDateTime}>
  <label for="recipients">Select Recipients:</label>
  <MultiSelect id="recipients" minSelect={1} bind:selected let:idx let:option options={recipients}>
    <span>
      {option.label}
      <span style:background={option.label} style=" width: 1em; height: 1em;" />
    </span>
  </MultiSelect>

  <label for="announcementText">Announcement Text:</label>
  <textarea id="announcementText" bind:value={announcementText} placeholder="Enter your announcement"></textarea>

  <button type="button" on:click={sendAnnouncement}>Send Announcement</button>

  <ul>
    {#each announcements as ann}
    <li>{ann.announcement_id}- scheduled time: {ann.scheduled_time}</li>
    {/each}
    </ul>
</form>


<style>
  form {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin: 2rem;
  }

  label {
    font-weight: bold;
    margin-bottom: 0.5rem;
  }
  select {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    margin-bottom: 1rem;
  }

  textarea {
    width: 100%;
    padding: 1rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    margin-bottom: 1rem;
  }

  button {
    padding: 0.5rem 1rem;
    background-color: #007bff;
    color: #fff;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.2s;
  }

  button:hover {
    background-color: #0056b3;
  }
</style>
