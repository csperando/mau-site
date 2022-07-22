let mauHeader = {
    template: "#mau-header-component",
    props: []
}


let mauStats = {
    template: "#mau-stats-component",
    props: ["games"],
    methods: {
        getWins: function() {
            try {
                var wins = this.games.map((obj) => {
                    return obj.winner.name;
                });

                var ty = wins.filter((el) => {
                    return el === "Ty";
                });

                var coleman = wins.filter((el) => {
                    return el == "Coleman";
                });

                return { "Ty": ty.length, "Coleman": coleman.length };

            } catch(error) {
                return {};

            }
        },
        getWinPercent: function() {
            try {
                var total = this.games.length;
                var wins = this.getWins();
                var ty = Math.floor(100 * wins.Ty / total);
                var coleman = Math.floor(100 * wins.Coleman / total);
                return { "Ty": `${ty}%`, "Coleman": `${coleman}%` };

            } catch(error) {
                return {"error": error.message};

            }
        }
    },
    computed: {
        stats: function() {
            return {
                "wins": this.getWins(),
                "winPercent": this.getWinPercent()
            };
        }
    }
}


let dashboard = {
    template: "#dashboard-component",
    props: [],
    data: function() {
        return {
            "games": []
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
