<script>
    import Sparkline from "sparklines"
    import items from "./yeni"

    let div
    let data = items.reverse()
    let canvas
    let index = null
    let current
    let clientX

    let options = {
        minValue: 1,
        maxValue: 14,
        lineWidth: 0,
        dotRadius: 10,
        endColor: 'transparent',
        minColor: 'transparent',
        maxColor: 'red'
    }

    const formatDate = date => {
        const [day, month, year] = date.split('.')
        return new Intl.DateTimeFormat('tr-TR', {
            dateStyle: 'long'
        }).format(new Date(`${year}-${month}-${day}`))
    }

    $: if (div) {
        canvas = new Sparkline(div, {
            ...options,
            width: window.innerWidth + 18,
            height: window.innerHeight / 2,
        })

        canvas.draw(
            data.reduce((acc, current) => {
                return [...acc, parseFloat(current.now.replace(',', '.'))]
            }, [])
            // data.reduce((acc, current) => {
            //     return [...acc, acc.length <= index ? parseFloat(current.now.replace(',', '.')) : 0]
            // }, [])
        )
    }

    $: if (index !== null) {
        current = data[index]
    }

    $: if (canvas) {
        function mousemove(e) {
            const percent = (e.clientX / (window.innerWidth - 1)) * 100
            const currentIndex = Math.ceil((percent * data.length) / 100)
            index = currentIndex > 0 ? currentIndex - 1 : 0
            clientX = e.clientX
        }

        function touchmove(e) {
            const percent = (e.touches[0].clientX / (window.innerWidth - 1)) * 100
            const currentIndex = Math.ceil((percent * data.length) / 100)
            index = currentIndex > 0 ? currentIndex - 1 : 0
            clientX = e.touches[0].clientX
        }

        function resize() {
            canvas = new Sparkline(div, {
                ...options,
                width: window.innerWidth + 18,
                height: window.innerHeight / 2,
            })
        }

        document.addEventListener('mousemove', mousemove)
        document.addEventListener('touchmove', touchmove)
        window.addEventListener('resize', resize)
    }

</script>

<div class="container">

    {#if clientX}
        <div class="line" style={`--position: ${clientX}px`}></div>
    {/if}

    <div class="headline">
        {#if current}
            <h1>{formatDate(current.date)}</h1>
            <p>
                1 USD = {current.now} TRY
            </p>
        {:else}
            <h1>{window.innerWidth < 800 ? 'Parmağını sürükle' : 'Farenizi oynatın'}!</h1>
        {/if}
    </div>
    <div bind:this={div} style={`--width: ${clientX + 9}px`} class="sparkline"></div>
</div>

<style>
    .container {
        height: 100%;
        display: flex;
        flex-direction: column;
        position: relative;
        overflow: hidden;
    }

    .container .line {
        width: 1px;
        height: 100%;
        background: red;
        position: absolute;
        top: 0;
        left: 0;
        z-index: 1;
        transform: translateX(var(--position));
    }

    .container > div {
        height: 50%;
    }

    .sparkline {
        position: relative;
        left: -9px;
        bottom: -9px;
        overflow: hidden;
        width: var(--width)
    }

    .headline {
        display: flex;
        align-items: center;
        justify-content: center;
        flex-direction: column;
        position: relative;
        z-index: 2;
        text-align: center;
    }

    .headline h1 {
        font-size: 10vh;
    }

    .headline p {
        font-size: 5vh;
        margin-top: 1rem;
        opacity: .7;
    }

    @media (max-width: 700px) {
        .headline h1 {
            font-size: 6vh;
        }

        .headline p {
            font-size: 2.5vh;
        }
    }
</style>