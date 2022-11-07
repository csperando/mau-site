let login = {
    template: "#mau-login-component",
    props: [],
    data: function() {
        return {
            username: null,
            password: null
        }
    },
    methods: {
        loginUser: function() {
            this.$emit("authStart");

            let req = {
                username: this.username,
                password: this.password
            };

            const loginReq = fetch("https://mau-rest.herokuapp.com/auth/login",
            {
                method: "POST",
                mode: "cors",
                headers: {
                    "Content-Type": "application/json",
                    "Accept": "application/json"
                },
                body: JSON.stringify(req)
            });
            loginReq.then((res) => {
                return res.json();

            }).then((res) => {
                if(res.statusCode == 200) {
                    this.$emit("loggedIn", res.data);
                } else {
                    alert(res.message);
                }

            }).catch((err) => {
                console.error(err);

            }).finally(() => {
                this.$emit("authEnd");

            });
        }
    }
}


let signup = {
    template: "#mau-signup-component",
    props: [],
    data: function() {
        return {
            firstName: null,
            lastName: null,
            email: null,
            username: null,
            password: null,
            confirm: null
        }
    },
    methods: {
        signupUser: function() {
            this.$emit("authStart");

            let req = {
                firstName: this.firstName,
                lastName: this.lastName,
                email: this.email,
                username: this.username,
                password: this.password,
                confirm: this.confirm
            }
            if(req.password != req.confirm) {
                alert("passwords do not match")
                this.$emit("authEnd");
                return;
            }

            const signupReq = fetch("https://mau-rest.herokuapp.com/auth/signup",
            {
                method: "POST",
                mode: "cors",
                headers: {
                    "Content-Type": "application/json",
                    "Accept": "application/json"
                },
                body: JSON.stringify(req)
            });
            signupReq.then((res) => {
                return res.json();

            }).then((res) => {
                alert(res.message);

            }).catch((error) => {
                console.error(error);

            }).finally(() => {
                this.$emit("authEnd");

            });
        }
    }
}


let userHeader = {
    template: "#mau-user-header-component",
    props: ["user"]
}


let settings = {
    template: "#mau-settings-component",
    props: []
}


let newRound = {
    template: "#mau-round-component",
    props: ["round", "index"],
    methods: {
        deleteRound: function() {
            this.$emit("deleteRound", this.round);
        }
    }
}


let newGame = {
    template: "#mau-new-game-component",
    props: [],
    data: function() {
        return {
            "game": {
                winner: {"name": ""},
                loser: {"name": ""},
                overall: [null, null],
                "rounds": []
            }
        }
    },
    components: {
        "new-round": newRound
    },
    methods: {
        newRound: function() {
            let prevIndex = this.game.rounds.length;
            this.game.rounds.push({
                winner: {"name": ""},
                loser: {"name": ""},
                points: null
            });
        },
        deleteRound: function(r) {
            let index = this.game.rounds.indexOf(r);
            let rounds = this.game.rounds.splice(index, 1);
        },
        submitGame: function() {
            this.$emit("loadingStart");

            let body = this.game;
            body.players = [this.game.winner, this.game.loser];
            let overall = parseInt(body.overall[0]) + parseInt(body.overall[1]);
            let roundTotal = 0;
            for(var r of body.rounds) {
                roundTotal += parseInt(r.points);
            }

            if(overall != roundTotal) {
                alert(`point error: round total ${roundTotal} does not equal overall total ${overall}`);
                this.$emit("loadingEnd");
                return;
            }

            const req = fetch("https://mau-rest.herokuapp.com/mau", {
                method: "POST",
                mode: "cors",
                headers: {
                    "Content-Type": "application/json",
                    "Accept": "application/json"
                },
                body: JSON.stringify(body)
            });
            req.then((res) => {
                return res.json();

            }).then((res) => {
                if(res.statusCode == 200) {
                    console.log("successfully posted new game");
                } else {
                    alert("error: " + res.message);
                }

            }).catch((error) => {
                console.error(error);

            }).finally(() => {
                this.$emit("loadingEnd");
                this.reset();

            });
        },
        reset: function() {
            this.game = {
                winner: {"name": ""},
                loser: {"name": ""},
                overall: [null, null],
                "rounds": []
            };
        }
    }
}


let menu = {
    template: "#mau-menu-component",
    props: ["user"],
    data: function() {
        return {
            "activeForm": null,
            "loading": false
        }
    },
    components: {
        "login": login,
        "signup": signup,
        "user-header": userHeader,
        "settings": settings,
        "new-game": newGame
    },
    methods: {
        toggleLogin: function() {
            this.activeForm = (this.activeForm == "login") ? null : "login";
        },
        toggleSignup: function() {
            this.activeForm = (this.activeForm == "signup") ? null : "signup";
        },
        toggleSettings: function() {
            this.activeForm = (this.activeForm == "settings") ? null : "settings";
        },
        toggleNewGame: function() {
            this.activeForm = (this.activeForm == "newGame") ? null : "newGame";
        },
        showLoader: function() {
            this.loading = true;
        },
        hideLoader: function() {
            this.loading = false;
        },
        loggedIn: function(user) {
            this.$emit("loggedIn", user);
        },
        logout: function() {
            document.cookie = "u=; expires=Thu, 01 Jan 1970 00:00:00 UTC;";
            window.location.reload();
        }
    },
    computed: {
        loggedOut: function() {
            return this.user == null;
        },
        isAdmin: function() {
            try {
                return this.user.roles == 'write';
            } catch(e) {
                return false;
            }
        }
    }
}


let modalView = {
    template: "#mau-modal-view-component",
    props: ["view", "user"],
    components: {
        "new-game": newGame,
        "mau-menu": menu
    },
    methods: {
        loggedIn: function(user) {
            this.$emit("loggedIn", user);
        }
    }
}


let modal = {
    template: "#mau-modal-component",
    props: ["modalActive", "user"],
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
        },
        loggedIn: function(user) {
            this.$emit("loggedIn", user);
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
    props: ["games", "loadingStats"],
    methods: {
        getWins: function() {
            try {
                var wins = this.games.map((obj) => {
                    return obj.winner.name.trim();
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
                        if(round.winner.name.trim() == "Ty") {
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
                        array.push((round.winner.name.trim() === "Ty") ? round.points : -1 * round.points);
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
            "modalActive": false,
            "loadingStats": true,
            "user": null
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
            this.loadingStats = false;

        }).catch((error) => {
            console.error(error);

        });

        // login user on page load using cookies
        try {
            let cookie = document.cookie;
            let uid = cookie.split(";").map((el) => {
                let keyValue = el.split("=");
                if(keyValue[0] == "u") {
                    return {"u": keyValue[1]};
                }
            })[0];
            uid = ("u" in uid) ? uid.u : "";

            const u = fetch("https://mau-rest.herokuapp.com/auth/login/" + uid);
            u.then((res) => {
                return res.json();

            }).then((res) => {
                if(res.statusCode == 200) {
                    this.user = res.data;
                }

            }).catch((error) => {
                console.error(error);

            });

        } catch(e) {
            // console.error(e.message);

        }

    },
    methods: {
        toggleModal: function(payload) {
            this.modalActive = !this.modalActive
        },
        hideModal: function(payload) {
            this.modalActive = false;
        },
        loggedIn: function(user) {
            this.user = user;
            document.cookie = `u=${user.id};`;
        }
    }
}


var app = new Vue({
    el: "#app",
    components: {
        "dashboard": dashboard
    }
});
