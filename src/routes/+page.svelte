<script>
    import CircleOfFifths from '../lib/circle-of-fifths.svelte';
</script>

<main>
    <h1>Interactive Circle of Fifths</h1>
    <CircleOfFifths />
  </main>
  
  <style>
    /* Ensure the entire viewport is black */
    html, body {
      margin: 0;
      padding: 0;
      background-color: black; /* Set the background color */
      height: 100%; /* Ensure the height covers the full viewport */
    }
  
    /* Style the main content */
    main {
      text-align: center;
      font-family: Arial, sans-serif;
      padding: 20px;
      background-color: black;
      min-height: 100vh; /* Ensure the main content covers the full viewport height */
      display: flex; /* Center content */
      flex-direction: column;
      justify-content: center;
      align-items: center;
    }
  
    /* Headings */
    h1 {
      color: white; /* White text for better contrast */
      font-size: 2vw;
    }
  </style>
  
