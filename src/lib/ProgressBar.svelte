<script>
    export let progress = 0;

    let left;
    let overallProgressWidth = 350;
    $: if (window.innerWidth > 1000) overallProgressWidth = 500;

    $: left = -70 + overallProgressWidth / (100 / progress)
     
</script>

<div class="progressBar-wrap">
    <div class='progressBar-title'>
        Выполнено {progress}%
    </div>
    <div class='progressBar-radius'>
        <div class='progressBar-active'>

        </div>
        <div class='progressBar-hold' style={`left: ${left}px`}>

        </div>
    </div>
</div>

<style>
    .progressBar-wrap {
        width: 100%;
        height: 35px;
        display: flex;
        position: relative;
        align-items: center;
        justify-content: center;
        z-index: 1;
        margin-top: 10px;
    }

    .progressBar-radius {
        overflow: hidden;
        width: 100%;
        height: 100%;
        border: 3px solid;
        border-radius: 99px;
        border-color: rgba(59, 75, 159, 1);
    }

    .progressBar-active {
        background: linear-gradient(356deg, #364297 16.59%, #6882F5 88.63%);
        width: 100%;
        height: 100%;
        transition: 1s;
    }

    .progressBar-hold {
        transition: 1s;
        height: 1000px;
        width: 1000px;
        background-color: #364297;
        position: relative;
        top: -110px;
        left: 0;
        rotate: 10deg;
    }
    .progressBar-title {
        color: white;
        position: absolute;
        font-size: 13px;
        z-index: 1;
        font-weight: 400;
    }

    @media screen and (min-width: 1200px) {
        .progressBar-wrap {
            width: 100%;
            height: 50px;
            display: flex;
            position: relative;
            align-items: center;
            justify-content: center;
            margin-top: 0px;
        }

        .progressBar-title {
            font-size: 21px;
            z-index: 1;
        }
        .progressBar-radius {
            width: 575px;
            border: 5px solid;
            border-radius: 99px;
            border-color: rgba(59, 75, 159, 1);
        }
    }

</style>