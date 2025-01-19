.. _display-graphs:

Graph Component
===============

.. seo::
    :description: Instructions for displaying graphs in ESPHome.
    :image: chart-line.svg

You can display a graph of a sensor value(s) using this component. The states used for the graph are stored in
memory at the time the sensor updates and will be lost when the device reboots.

Examples:

.. figure:: images/display_rendering_graph.png
    :align: center

Graph component with options for grids, border and line-types.

.. code-block:: yaml

    graph:
      # Show bare-minimum auto-ranged graph
      - id: single_temperature_graph
        sensor: my_temperature
        duration: 1h
        width: 151
        height: 51
      # Show multi-trace graph
      - id: multi_temperature_graph
        duration: 1h
        x_grid: 10min
        y_grid: 1.0     # degC/div
        width: 151
        height: 51
        traces:
          - sensor: my_inside_temperature
            line_type: DASHED
            line_thickness: 2
            color: my_red
          - sensor: my_outside_temperature
            line_type: SOLID
            continuous: true
            line_thickness: 3
            color: my_blue
          - sensor: my_beer_temperature
            line_type: DOTTED
            line_thickness: 2
            color: my_green

Configuration variables:
------------------------

- **id** (**Required**, :ref:`config-id`): The ID with which you will be able to reference the graph later
  in your display code.
- **width** (**Required**, int): The graph width in pixels
- **height** (**Required**, int): The graph height in pixels
- **duration** (**Required**, :ref:`config-time`): The total graph history duration.
- **border** (*Optional*, boolean): Specifies if a border will be drawn around the graph. Default is True.
- **x_grid** (*Optional*): Specifies the time per division. If not specified, no vertical grid will be drawn.
- **y_grid** (*Optional*, float): Specifies the number of units per division. If not specified, no horizontal grid will be drawn.
- **max_range** (*Optional*): Specifies the maximum Y-axis range.
- **min_range** (*Optional*): Specifies the minimum Y-axis range.
- **max_value** (*Optional*): Specifies the maximum Y-axis value.
- **min_value** (*Optional*): Specifies the minimum Y-axis value.
- **traces** (*Optional*): Use this to specify more than a single trace.

Trace specific fields:

- **sensor** (*Optional*, :ref:`config-id`): The sensor value to plot
- **line_thickness** (*Optional*): Defaults to 3
- **line_type** (*Optional*): Specifies the plot line-type. Can be one of the following: ``SOLID``, ``DOTTED``, ``DASHED``. Defaults to ``SOLID``.
- **continuous** (*Optional*): connects the individual points to make a continuous line.  Defaults to ``false``.
- **color** (*Optional*): Sets the color of the sensor trace.

And then later in code:

.. code-block:: yaml

    display:
      - platform: ...
        # ...
        pages:
          - id: page1
            lambda: |-
        pages:
          - id: page1
            lambda: |-
              // Draw the graph at position [x=10,y=20]
              it.graph(10, 20, id(single_temperature_graph));
          - id: page2
            lambda: |-
              // Draw the graph at position [x=10,y=20]
              it.graph(10, 20, id(multi_temperature_graph), my_yellow);

    color:
      - id: my_red
        red: 100%
        green: 0%
        blue: 0%
      - id: my_green
        red: 0%
        green: 100%
        blue: 0%
      - id: my_blue
        red: 0%
        green: 0%
        blue: 100%
      - id: my_yellow
        red: 100%
        green: 100%
        blue: 0%
.. note::

    Here are some things to note:
    - Setting ``y_grid`` will expand any specified range to the nearest multiple of grid spacings.
    - Axis labels are currently not possible without manually placing them.
    - The grid and border color is set with it.graph(), while the traces are defined separately.

Advanced usage:
---------------
The graphing object already does a lot of work to automatically scale the graph to the values displayed on it.  
If you want to hook in to this to for example print axis values, you can get the current limits by calling:

.. code-block:: yaml
    id(my_graph).get_value_min()
    id(my_graph).get_value_max()

Here's an example using a Waveshare 2.13" E-Paper display

.. code-block:: yaml
    graph:
      - id: my_graph
        height: 61
        width: 220
        duration: 
          days: 0
          hours: 0
          minutes: 5
        border: True
        y_grid: 100
        min_value: 0
        traces:
          - sensor: sens
            line_type: SOLID
            line_thickness: 2
            continuous: True
    
    font:
      - file:
          type: gfonts
          family: Literata
          weight: 400
        id: literata_tiny
        size: 13
    spi:
      clk_pin: GPIO13
      mosi_pin: GPIO14
    
    display:
      - platform: waveshare_epaper
        id: disp
        cs_pin: GPIO15
        dc_pin: GPIO27
        reset_pin: GPIO26
        busy_pin: GPIO25
        model: 2.13inv3
        full_update_every: 60
        rotation: 270
        update_interval: 5s
        pages: 
          - id: home_screen
            lambda: |-
              int y_start = 50;

              it.printf(it.get_width()-220, y_start, id(literata_tiny), TextAlign::CENTER_RIGHT, "%.0f", id(my_graph).get_value_max());
              it.printf(it.get_width()-220, y_start + 61, id(literata_tiny), TextAlign::CENTER_RIGHT, "%.0f", id(my_graph).get_value_min());
              it.graph(it.get_width()-220, y_start, id(my_graph));
              ESP_LOGD("custom", "%f, %f", id(my_graph).get_value_min(), id(my_graph).get_value_max());

See Also
--------
- :apiref:`graph/graph.h`