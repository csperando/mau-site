let newGame = {
    template: "#mau-new-game-component",
    props: []
}


let menu = {
    template: "#mau-menu-component",
    props: []
}


let modalView = {
    template: "#mau-modal-view-component",
    props: ["view"],
    components: {
        "new-game": newGame,
        "mau-menu": menu
    }
}


let modal = {
    template: "#mau-modal-component",
    props: ["modalActive"],
    components: {
        "modal-view": modalView
    }, data: function() {
        return {
            isActive: this.modalActive
        }
    },
    methods: {
        hideModal: function(event) {
            if(event.target.className == "dark-background") {
                this.$emit("hide-modal");
            }
        }
    }
}


let mauHeader = {
    template: "#mau-header-component",
    props: [],
    methods: {
        toggleModal: function() {
            this.$emit("toggle-modal", "menu");
        }
    }
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
        },
        getLongestStreak: function() {
            try{
                var ty = 0;
                var coleman = 0;

                for(var game of this.games) {
                    var streakC = 0;
                    var streakT = 0;

                    for(var round of game.rounds) {
                        if(round.winner.name == "Ty") {
                            streakT += 1;
                            ty = (streakT > ty) ? streakT : ty;
                            streakC = 0;
                        } else {
                            streakC += 1;
                            coleman = (streakC > coleman) ? streakC : coleman;
                        }
                    }
                }

                return { "Ty": ty, "Coleman": coleman };

            } catch(error) {
                // console.error(error);
                return { "Error": error.message };
            }
        },
        getSavage: function() {
            try {
                var array = [0];
                for(var game of this.games) {
                    for(var round of game.rounds) {
                        array.push((round.winner.name === "Ty") ? round.points : -1 * round.points);
                    }
                }

                var coleman = -1 * Math.min(...array);
                var ty = Math.max(...array);

                return { "Ty": ty, "Coleman": coleman };

            } catch(error) {

            }
        }
    },
    computed: {
        stats: function() {
            return {
                "wins": this.getWins(),
                "winPercent": this.getWinPercent(),
                "longestStreak": this.getLongestStreak(),
                "savage": this.getSavage()
            };
        }
    }
}


let dashboard = {
    template: "#dashboard-component",
    props: [],
    data: function() {
        return {
            "games": [],
            "modalActive": false
        }
    },
    components: {
        "mau-header": mauHeader,
        "mau-stats": mauStats,
        "modal": modal
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
        toggleModal: function(payload) {
            this.modalActive = !this.modalActive
        },
        hideModal: function(payload) {
            this.modalActive = false;
        }
    }
}


var app = new Vue({
    el: "#app",
    components: {
        "dashboard": dashboard
    }
});
