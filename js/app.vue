const games = fetch("https://mau-rest.herokuapp.com/mau");
games.then((res) => {
    return res.json();

}).then((res) => {

    let mauHeader = {
        template: "#mau-header-component",
        props: []
    }


    let dashboard = {
        template: "#dashboard-component",
        props: ["gameData"],
        components: {
            "mau-header": mauHeader
        }
    }


    var app = new Vue({
        el: "#app",
        data: {
            gameData: res
        },
        components: {
            "dashboard": dashboard
        }
    });

}).catch((error) => {
    // console.error(error);
    return [];

});
