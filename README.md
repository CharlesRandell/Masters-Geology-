Geological visualization of elements
====================================

The aim of this folder is to…. create a grouped box and whisker plot
with the ggplot2 package that shows a variety of trace element
concentrations in different forms of Carrollite in the central African
Copperbelt

load packages in R

    library(ggplot2) 
    pacman::p_load(tidyverse)
    library(wesanderson)

The use of ggplots2 to create the box and whisker The grouped plots that
will be used to test the geological data

loading the excel data via clipboard this will not work without it being
copied!!

    # copying the data in 

    #CarStrat <- read.table(pipe("pbpaste"), sep="\t", header = TRUE)

    #CarDissem <- read.table(pipe("pbpaste"), sep="\t", header = TRUE)

    #CarCross <- read.table(pipe("pbpaste"), sep="\t", header = TRUE)

    #CarJack <- read.table(pipe("pbpaste"), sep="\t", header = TRUE)

    # moving it to csv files for the knit function

    #write.csv(CarStrat,"/Users/charlesrandell/Other/Random projects/Box and Whisker/data/CarStrat.csv", row.names = FALSE)

    #write.csv(CarDissem,"/Users/charlesrandell/Other/Random projects/Box and Whisker/data/CarDissem.csv", row.names = FALSE)

    #write.csv(CarCross,"/Users/charlesrandell/Other/Random projects/Box and Whisker/data/CarCross.csv", row.names = FALSE)

    #write.csv(CarJack,"/Users/charlesrandell/Other/Random projects/Box and Whisker/data/CarJack.csv", row.names = FALSE)

    setwd("/Users/charlesrandell/Other/Random projects/Box and Whisker/data")

    CarStrat <- read.csv("CarStrat.csv", header = TRUE)

    CarDissem <- read.csv("CarDissem.csv", header = TRUE)

    CarCross <- read.csv("CarCross.csv", header = TRUE)

    CarJack <- read.csv("CarJack.csv", header = TRUE)

Tidyr
-----

Lets see what happens… Changing the data imported in wide format into a
long tidy format

    TidyCarCross <- CarCross %>% tidyr::gather(Header, val)

    ## Warning: attributes are not identical across measure variables;
    ## they will be dropped

    TidyCarDissem <- CarDissem %>% tidyr::gather(Header, val)

    ## Warning: attributes are not identical across measure variables;
    ## they will be dropped

    TidyCarStrat <- CarStrat  %>% tidyr::gather(Header, val)

    ## Warning: attributes are not identical across measure variables;
    ## they will be dropped

    TidyCarJack <- CarJack  %>% tidyr::gather(Header, val)

    ## Warning: attributes are not identical across measure variables;
    ## they will be dropped

dplyr
-----

This creates the new variable that has the extra column with the added
indicator to be merged to the final dataset. The use of the tidyverse to
convert the data into a long format rather that a wide so that ggplot2
can create the boxplot with it

    Tcarcross <- TidyCarCross %>% mutate(Classes = "4CarCross")

    Tcardissem <- TidyCarDissem %>% mutate(Classes = "2CarDissem")

    Tcarstrat <- TidyCarStrat %>% mutate(Classes = "1CarStrat")

    Tcarjack <- TidyCarJack %>% mutate(Classes = "3CarJack")

binding and filtering the data
------------------------------

Bind the datasets together Secondly the readings that are below the
detectable level are filtered out

    Final <- rbind.data.frame(Tcarstrat, Tcardissem, Tcarcross, Tcarjack)
    Final <- Final %>% filter(val != "Below LOD")
    Final <- Final %>% filter(Header != "Te_ppm_m125")

gsub to format elements
-----------------------

mutate function that can remove certain phrases within values in a
column

    Final <- Final %>% mutate(Header = gsub("_ppm", "", Header))
    Final <- Final %>% mutate(Header = gsub("_m126", "", Header))

Convert the variable Classes and Header from a character to a factor
variable Convert the value variable from a character to a numerical
variable - this is very important and the code will not run without it

    Final[, 'Classes'] <- as.factor(Final[, 'Classes'])
    Final[, 'val'] <- as.numeric(Final[, 'val'])
    Final[, 'Header'] <- as.factor(Final[, 'Header'])

Firstly the outliers are removed, any value that exceeds 150 is taked
out

    Final <- Final %>% filter(val < 150)

ggplot to create graphs
-----------------------

The plot is constructed using ggplot, with two iterations of the same
graph, the first shows the raw output on a linear scale And second has a
log scale y axis, along with updated labels and different colour palette

    # The Main ggplot

    sfplot <- ggplot(Final, aes(x=Header, y=val, fill=Classes)) + 
        geom_boxplot(outlier.size = 0.01) + stat_summary(fun = mean, shape = 4, aes(group=Classes), position=position_dodge(0.75), show.legend = FALSE, color="black", size=0.25)

    sfplot

    ## Warning: Removed 56 rows containing missing values (geom_segment).

![](README_files/figure-markdown_strict/unnamed-chunk-9-1.png)

    # For some reason this is what worked for the color codes to range from 2-5

    wes1 <- wes_palette("FantasticFox1")[2:5]
    roy2 <- wes_palette("Royal2")[1:3]


    # The Final Wes Anderson themed plot

    sfplot  +
    coord_trans(y = "log") + scale_y_continuous(breaks = c(0.01, 0.1, 1, 10, 100), labels = c(0.01, 0.1, 1, 10, 100)) + labs(x="Elements", y = "Log (Concentrations) / ppm") + theme(legend.position="bottom") + scale_fill_manual(guide = guide_legend(title = ""), breaks=c("1CarStrat", "2CarDissem", "3CarJack", "4CarCross"), values=c(wes1), labels=c("Stratiform Carrollite", "Disseminated Carrollite", "Jack Vein Carrollite", "Cross-Cutting Vein Carrollite")) + theme(axis.text.x = element_text(size=12, face = "bold"), axis.text.y = element_text(size=12, face = "bold"), legend.text=element_text(size=14), axis.title=element_text(size=15,face="bold"))

    ## Warning: Removed 56 rows containing missing values (geom_segment).

![](README_files/figure-markdown_strict/unnamed-chunk-9-2.png)