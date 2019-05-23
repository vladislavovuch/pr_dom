<template>
    <div class="app">
        <div class="search">
            <input type="text">
            <button>Search</button>
        </div>
        <div id="table">

        </div>
    </div>
</template>

<script>
    export default {
        mounted() {

            function changeRoutes(routes) {
                for (let i = 0; i < routes.length; i++) {
                    for (const key in routes[i]) {
                        routes[i][key] = {
                            text: routes[i][key],
                            marked: false,
                        }
                    }
                }
                return routes;
            }

            let routes = null;
            function createTableHeader(routes) {
                const tr = document.createElement('tr');
                for (const key in routes[0]) {
                    const th = document.createElement('th');
                    th.innerText = key;
                    th.dataset.field = key;
                    tr.appendChild(th);
                }
                return tr;
            }

            function createTableContent(routes, table) {
                for (let i = 0; i < routes.length; i++) {
                    const route = routes[i];
                    console.log(route);
                    const tr = document.createElement('tr');
                    for (const item in route) {
                        const td = document.createElement('td');
                        td.innerText = route[item].text;
                        if (route[item].marked) {
                            td.classList.add('marked')
                        }
                        tr.appendChild(td);
                    }
                    console.log(table);
                    table.appendChild(tr);
                }
            }

            function createTable(routes) {
                //get table
                const table = document.createElement('table');
                //create th
                const tableHeader = createTableHeader(routes);
                table.appendChild(tableHeader);

                // create tr + td
                createTableContent(routes, table);

                //add listener
                table.onclick = tableSort.bind(routes);

                //add created element into dom
                document.getElementById('table').appendChild(table);
            }

            function getRoutes() {
                fetch('http://localhost:3000/routes')
                    .then(response => {
                        console.log(response)
                        if (response.status !== 200) {
                            throw new Error(response.statusText);
                        }
                        return response.json();
                    })
                    .then(response => {
                        console.log(response);
                        response = changeRoutes(response);
                        routes = response;
                        createTable(response);

                    })
                    .catch(err => {
                        console.warn(err)
                    })
            }

            getRoutes();

            function reRenderTable(routes) {
                const div = document.getElementById('table');
                div.removeChild(div.firstChild);
                createTable(routes);
            }

            let lastSortedKey = null;

            function tableSort(event) {
                console.log(event);
                const key = event.target.dataset.field;
                console.log(key);
                if (key) {
                    if (key === lastSortedKey) {
                        this.reverse();
                    } else {
                        if (key === 'ticketCost') {
                            this.sort((route1, route2) => {
                                return +route1[key].text > +route2[key].text ? 1 : -1;
                            });
                        } else {
                            this.sort((route1, route2) => {
                                return route1[key].text > route2[key].text ? 1 : -1;
                            });
                        }
                    }

                    lastSortedKey = key;

                    reRenderTable(this)
                    console.log(this);
                }
            }

            document.querySelector('.search button').addEventListener('click', findRoutes);

            function findRoutes() {
                const value = document.querySelector('.search input').value;
                if (value) {
                    for (let i = 0; i < routes.length; i++) {
                        let fl = false;
                        for (const key in routes[i]) {
                            let result = routes[i][key].text.match(value);
                            if (result) {
                                routes[i][key].marked = true;
                            }
                        }
                    }

                    reRenderTable(routes);
                }
            }
        }
    }

</script>

<style >
    .app {
        width: 100%;
        display: flex;
        justify-content: center;
        flex-direction: column;
        align-items: center;
    }

    #table {
        max-width: 1140px;
        width: 100%;
    }

    .marked {
        background-color: yellow;
    }
    tr {
        width: 100%;
    }

    td {
        width: 100%;
        border: 1px solid black;
    }

    th {
        cursor: pointer;
    }
    table {
        width: 100%;


    }
</style>
