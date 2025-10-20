<script>
    let foods = [
        {name: 'Apple', image:'food/apple.png', calories: 105, fat:0.2, protein: 0.3, carbs: 24.4},
        {name: 'Banana', image:'food/banana.png', calories: 200, fat:0.7, protein: 2.5, carbs: 51},
        {name: 'Orange', image:'food/orange.png', calories: 81, fat:0.2, protein: 1.5, carbs: 21}
    ];

    let active = foods[0]
</script>

<div class="panel"> 
    <h2>Nutrition Analyzer</h2> 
    <select id="food" bind:value={active}>  
        {#each foods as food}
            <option value={food}>
                {food.name}
            </option>
        {/each}
    </select>
</div>

<img src={active.image} alt="selected-food" class="food-image"/>

<div class="nutrition-panel">
    <h1>Nutrition Values</h1>
    <p><b>Calories :</b> {active.calories}</p>
    <p><b>Fat :</b> {active.fat} grams</p>
    <p><b>Protein :</b> {active.protein} grams</p>
    <p><b>Carbs :</b> {active.carbs} grams</p>
</div>