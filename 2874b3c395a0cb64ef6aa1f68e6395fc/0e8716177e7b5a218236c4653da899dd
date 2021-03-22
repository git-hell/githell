require "kemal"
require "celestine"

require "./config"

svg = Celestine.draw do |ctx|
    ctx.width = WIDTH
    ctx.height = HEIGHT
    ctx.width_units = "px"
    ctx.height_units = "px"
    
    ctx.rectangle do |r|
        r.x = 0
        r.y = 0
        r.width = WIDTH
        r.height = HEIGHT

        r.animate do |a|
            a.duration = SUNRISE_DURATION
            a.duration_units = "s"
            a.attribute = "fill"
            a.values = [SUNRISE_START, SUNRISE_END] of SIFNumber
            a.freeze = true

            a
        end

        r
    end

    ctx.mask do |m|
        m.id ="sun1-mask"

        m.rectangle do |r|
            r.x = 0
            r.y = 0
            r.fill = "white"
            r.width = WIDTH
            r.height = HEIGHT

            r
        end

        m.rectangle do |r|
            r.x = 0
            r.y = HEIGHT / 2
            r.fill = "black"
            r.width = WIDTH
            r.height = HEIGHT / 2
            
            r
        end

        m
    end

    ctx.mask do |m|
        m.id ="sun2-mask"

        m.rectangle do |r|
            r.x = 0
            r.y = 0
            r.fill = "white"
            r.width = WIDTH
            r.height = HEIGHT

            r
        end

        m.rectangle do |r|
            r.x = 0
            r.y = 0
            r.fill = "black"
            r.width = WIDTH
            r.height = HEIGHT / 2
            
            r
        end

        m
    end

    ctx.filter do |f|
        f.id = "reflection"
        
        f.turbulence do |t|
            t.base_freq = 0.01
            t.num_octaves = 10

            t.animate do |a|
                a.duration = 20
                a.duration_units = "s"
                a.attribute = "seed"
                a.from = 0
                a.to = 100
                a.repeat_count = "indefinite"
                a
            end
            
            t
        end

        f.displacement_map do |d|
            d.input = "SourceGraphic"
            d.scale = 12 
            d
        end

        f
    end

    ctx.circle do |c|
        c.x = WIDTH / 2
        c.radius = SUN_RADIUS
        c.fill = SUN_COLOR
        c.set_mask "sun1-mask"

        c.animate do |a|
            a.duration = SUNRISE_DURATION
            a.duration_units = "s"
            a.attribute = "cy"
            a.from = HEIGHT / 2 - SUNRISE_START_POS
            a.to = SUNRISE_END_POS
            a.freeze = true

            a
        end

        c
    end

    ctx.circle do |c|
        c.x = WIDTH / 2
        c.radius = SUN_RADIUS
        c.fill = SUN_COLOR
        c.set_mask "sun2-mask"

        c.set_filter "reflection"

        c.animate do |a|
            a.duration = SUNRISE_DURATION
            a.duration_units = "s"
            a.attribute = "cy"
            a.from = HEIGHT / 2 + SUNRISE_START_POS
            a.to = HEIGHT - SUNRISE_END_POS
            a.freeze = true

            a
        end

        c
    end

    ctx.path do |p|
        p.stroke = "#000000"
        p.stroke_width = 1
        p.fill_opacity = 0

        p.a_move WIDTH / 2, HEIGHT / 2
        p.r_move -ISLAND_WIDTH / 2, 0
        p.r_q_bcurve ISLAND_WIDTH / 2, -ISLAND_HEIGHT, ISLAND_WIDTH, 0 
        p.a_move WIDTH / 2, HEIGHT / 2
        p.r_move 0, -ISLAND_HEIGHT / 2
        p.r_line 0, -TRUNK_HEIGHT
        
        LEAF_COUNT.times do |n|
            p.r_line -LEAF_WIDTH * (n + 1), -LEAF_HEIGHT
            p.r_line LEAF_WIDTH * (n + 1) * 2, 0
            p.r_line -LEAF_WIDTH * (n + 1), LEAF_HEIGHT
            p.r_move 0, -LEAF_HEIGHT
        end
        LEAF_COUNT.times do |n|
            n = LEAF_COUNT - n
            p.r_line -LEAF_WIDTH * n, 0
            p.r_line LEAF_WIDTH * n, -LEAF_HEIGHT
            p.r_line LEAF_WIDTH * n, LEAF_HEIGHT
            p.r_line -LEAF_WIDTH * n, 0
            p.r_move 0, -LEAF_HEIGHT
        end
        p.close

        p
    end

    ctx.path do |p|
        p.stroke = "#000000"
        p.stroke_width = 1
        p.fill_opacity = 0

        p.set_filter "reflection"

        p.a_move WIDTH / 2, HEIGHT / 2
        p.r_move -ISLAND_WIDTH / 2, 0
        p.r_q_bcurve ISLAND_WIDTH / 2, ISLAND_HEIGHT, ISLAND_WIDTH, 0 
        p.a_move WIDTH / 2, HEIGHT / 2
        p.r_move 0, ISLAND_HEIGHT / 2
        p.r_line 0, TRUNK_HEIGHT
        
        LEAF_COUNT.times do |n|
            p.r_line -LEAF_WIDTH * (n + 1), LEAF_HEIGHT
            p.r_line LEAF_WIDTH * (n + 1) * 2, 0
            p.r_line -LEAF_WIDTH * (n + 1), -LEAF_HEIGHT
            p.r_move 0, LEAF_HEIGHT
        end
        LEAF_COUNT.times do |n|
            n = LEAF_COUNT - n
            p.r_line -LEAF_WIDTH * n, 0
            p.r_line LEAF_WIDTH * n, LEAF_HEIGHT
            p.r_line LEAF_WIDTH * n, -LEAF_HEIGHT
            p.r_line -LEAF_WIDTH * n, 0
            p.r_move 0, LEAF_HEIGHT
        end
        p.close

        p
    end

    ctx.path do |p|
        p.stroke = "#000000"
        p.stroke_width = 2

        p.a_move 0, HEIGHT / 2
        p.r_line WIDTH, 0
        p.close
        p
    end
end

get "/" do
    <<-EOF
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8">
            <title>Sunrise over Island</title>
            <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
            <style>
                body {
                    position: absolute;
                    top: 0;
                    left: 0;
                    width: 100vw;
                    height: 100vh;
                    padding: 0;
                    margin: 0;
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    flex-direction: column;
                }
            </style>
        </head>
        <body>
            #{svg}
        </body>
    </html>  
    EOF
end

Kemal.run
