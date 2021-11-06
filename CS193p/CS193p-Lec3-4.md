### CS193p Lec3-4

#### Lecture 3: MVVM and the Swift type system

* MVVM Architecture

    ![4.jpg](./images/4.jpg)

* Swift Type System

    * struct
    * class
    * protocal
    * type
    * enum
    * functions

* Differences between struct and class

    ![5.jpg](./images/5.jpg)
    



#### Lecture 4: Memorize Game Logic

* enum

    * Declaring enum: another variety of data structure

        ![6.jpg](./images/6.jpg)

    * Associated Data

        ![7.jpg](./images/7.jpg)

    * Setting the value of an enum
    
        ![8.jpg](./images/8.jpg)

    * Using switch
    
        ![9.jpg](./images/9.jpg)
        
        ![10.jpg](./images/10.jpg)
        
        ```swift
        /// switch supporting other kinds of data
        let s: String = "hello"
        switch s {
        		case "hello":
          			print("hello")
          			print("hello again")
            case "goodbye":
          			print("goodbye")
            default:
          			print("default")
        }
        ```
        
    * enum can have methods and vars
    
        ![11.jpg](./images/11.jpg)
    
* Optionals

    Just an enum

    ![12.jpg](./images/12.jpg)

    ![13.jpg](./images/13.jpg)
    
    An elegant way to access an optional variable
    
    ![14.jpg](./images/14.jpg)
    
    understanding syntax ??
    
    ![15.jpg](./images/15.jpg)
    
    optional chain
    
    ![16.jpg](./images/16.jpg)

