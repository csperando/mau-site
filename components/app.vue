let mauHeader = {
    template: "#mau-header-component",
    props: []
}


let mauStats = {
    template: "#mau-stats-component",
    props: ["games"],
    computed: {
        wins: function() {
            try {
                var wins = this.games.map((obj) => {
                    return obj.winner.name;
                });
                var ty = wins.filter((el) => {
                    return el === "Ty";
                });
                var coleman = wins.filter((el) => {
                    return el == "Coleman";
                })
                return {
                    "Ty": ty.length,
                    "Coleman": coleman.length
                }

            } catch(error) {

            }
        },
        winPercent: function() {
            try {
                var total = this.games.length;
                var ty = Math.floor(100 * total / this.wins.ty);
                var coleman = Math.floor(100 * total / this.wins.coleman);
                return {
                    "Ty": ty,
                    "Coleman": coleman
                }

            } catch(error) {

            }
        }
    }
}


let dashboard = {
    template: "#dashboard-component",
    props: [],
    data: function() {
        return {
            "games": {}
        }
    },
    components: {
        "mau-header": mauHeader,
        "mau-stats": mauStats
    },
    created: function() {
        const g = fetch("https://mau-rest.herokuapp.com/mau");
        g.then((res) => {
            return res.json();

        }).then((res) => {
            this.games = res;

        }).catch((error) => {
            console.error(error);

        });
    },
    methods: {

    }
}


var app = new Vue({
    el: "#app",
    components: {
        "dashboard": dashboard
    }
});
