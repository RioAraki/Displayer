1. create a manual editor which can
    1. add node and edge
        1. edit node name and edge name
    2. node
        1. usually a service, can have diff type
            1. name: name of the service
            2. type: C++ backend service/ C# GUI
            3. description: what does it do?
            4. connection: one upstream per connection? how data transmission works? subscription base? request base? etc.
            5. link: link to doc? link to optic?
        2. or a middleware? redis?
    3. edge
        1. usually data transmission
            1. name: what is the data being transferred
            2. transmit media: kafka? redis? https?
            3. transmit rule? subscription? request?
    4. persist data, load data
    5. how to edit? how to review?