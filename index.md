---
title: "Research Trends Code"
output: html_notebook
---

This is the code and bibtex files for the paper Research trends in Coquerel’s sifaka (Propithecus coquereli)

```{r include=FALSE}
using<-function(...) {
    libs<-unlist(list(...))
    req<-unlist(lapply(libs,require,character.only=TRUE))
    need<-libs[req==FALSE]
    n<-length(need)
    if(n>0){
        libsmsg<-if(n>2) paste(paste(need[1:(n-1)],collapse=", "),",",sep="") else need[1]
        print(libsmsg)
        if(n>1){
            libsmsg<-paste(libsmsg," and ", need[n],sep="")
        }
        libsmsg<-paste("The following packages could not be found: ",libsmsg,"\n\r\n\rInstall missing packages?",collapse="")
        if(winDialog(type = c("yesno"), libsmsg)=="YES"){       
            install.packages(need)
            lapply(need,require,character.only=TRUE)
        }
    }
}
using("bibliometrix","tidyverse")
```


```{r include=FALSE}
#read in data

M <-
  convert2df(
    "https://raw.githubusercontent.com/funkepunkemonke/Research-trends-in-Propithecus-coquereli/main/savedrecs-PropithecusCoquereli.bib",
    dbsource = "isi",
    format = "bibtex"
  )
allM <-
  convert2df(
    "https://raw.githubusercontent.com/funkepunkemonke/Research-trends-in-Propithecus-coquereli/main/savedrecs-allPropithecus.bib",
    dbsource = "isi",
    format = "bibtex"
  )
lemur1M <-
  convert2df(
    "https://raw.githubusercontent.com/funkepunkemonke/Research-trends-in-Propithecus-coquereli/main/savedrecs-lemur1.bib",
    dbsource = "isi",
    format = "bibtex"
  )
lemur2M <-
  convert2df(
    "https://raw.githubusercontent.com/funkepunkemonke/Research-trends-in-Propithecus-coquereli/main/savedrecs-lemur2.bib",
    dbsource = "isi",
    format = "bibtex"
  )
lemur3M <-
  convert2df(
    "https://raw.githubusercontent.com/funkepunkemonke/Research-trends-in-Propithecus-coquereli/main/savedrecs-lemur3.bib",
    dbsource = "isi",
    format = "bibtex"
  )
lemur4M <-
  convert2df(
    "https://raw.githubusercontent.com/funkepunkemonke/Research-trends-in-Propithecus-coquereli/main/savedrecs-lemur4.bib",
    dbsource = "isi",
    format = "bibtex"
  )
lemur5M <-
  convert2df(
    "https://raw.githubusercontent.com/funkepunkemonke/Research-trends-in-Propithecus-coquereli/main/savedrecs-lemur5.bib",
    dbsource = "isi",
    format = "bibtex"
  )
lemur6M <-
  convert2df(
    "https://raw.githubusercontent.com/funkepunkemonke/Research-trends-in-Propithecus-coquereli/main/savedrecs-lemur6.bib",
    dbsource = "isi",
    format = "bibtex"
  )
lemurs <-
  plyr::rbind.fill(lemur1M, lemur2M, lemur3M, lemur4M, lemur5M, lemur6M)
```


```{r}
#Descriptive analysis
results <- biblioAnalysis(M, sep = ";")
S <- summary(object = results, k = 10, pause = FALSE)
knitr::kable(S$MainInformationDF, caption = "Summary Information") #main information
knitr::kable(S$MostProdAuthors, caption = "Most Productive Authors") #Most productive Authors
knitr::kable(S$MostCitedPapers, caption = "Most Cited Papers") #most cited paper
plot(x = results, k = 10, pause = FALSE)
```


```{r}
#Top-Authors’ Productivity over the Time:
topAU <- authorProdOverTime(M, k = 10, graph = TRUE)
```


```{r}
#Chart of Paper per decade
G <- M %>%
  mutate(Decade = as.numeric(PY) - as.numeric(PY) %% 10) %>%
  group_by(Decade) %>%
  summarize(val = n()) %>%
  ungroup()

G$Decade = as.factor(G$Decade)

ggplot2::ggplot(G, aes(x = Decade, y = val)) +
  geom_bar(stat = "identity", aes(fill = factor(Decade))) +
  scale_x_discrete(labels = G %>% distinct(Decade) %>% mutate(Decade = paste0(Decade, "s")) %>% pull()) +
  geom_text(aes(label = format(val, big.mark = ",")), size = 5, vjust =
              -0.3) +
  ggtitle('Number of Papers per Decade') +
  theme(
    panel.border = element_blank(),
    axis.title.x = element_blank(),
    axis.title.y = element_blank(),
    legend.position = "none",
    plot.title = element_text(face = "bold", size = 18, hjust = 0.5),
    text = element_text(),
    panel.background = element_rect(fill = "white"),
    plot.background = element_rect(colour = NA),
    axis.title = element_text(face = "bold", size = rel(1)),
    axis.text = element_text(size = 16),
    axis.line = element_line(colour = "black"),
    panel.grid.major = element_line(colour = "#f0f0f0"),
    panel.grid.minor = element_blank(),
    axis.ticks = element_line(colour = "black"),
    plot.margin = unit(c(10, 5, 5, 5), "mm"),
    strip.background = element_rect(colour = "#f0f0f0", fill = "#f0f0f0"),
    strip.text = element_text(face = "bold")
  )
```


```{r}
#Co-word analysis: cluster terms extracted from keywords, titles, or abstracts
NetMatrix <-
  biblioNetwork(M,
                analysis = "co-occurrences",
                network = "keywords",
                sep = ";")
netplot = networkPlot(
  NetMatrix,
  normalize = "association",
  weighted = T,
  n = 50,
  Title = "Keyword Co-occurrences",
  type = "auto",
  cluster = "louvain",
  community.repulsion = 0.15,
  size = T,
  edgesize = 7,
  labelsize = 1,
  remove.multiple = TRUE,
  remove.isolates = T
)
```


```{r}
#perform multiple correspondence analysis (MCA): identify clusters of documents that express common concepts
CS <-
  conceptualStructure(
    M,
    field = "ID",
    method = "MCA",
    minDegree = 5,
    clust = 4 ,
    k.max = 5,
    stemming = FALSE,
    labelsize = 15
  )
```


```{r}
#trend topics by year
res <-
  fieldByYear(
    M,
    field = "ID",
    timespan = c(1978, 2022),
    min.freq = 5,
    n.items = 5,
    graph = TRUE
  )
```


```{r}
#Thematic Map -  starts from a co-occurrence keyword network to plot in a two-dimensional map the themes of a domain.
Map = thematicMap(
  M,
  field = "ID",
  n = 55,
  minfreq = 4,
  stemming = TRUE,
  size = 0.7,
  n.labels = 3
)
plot(Map$map)
```


```{r}
Clusters = Map$words[order(Map$words$Cluster, -Map$words$Occurrences), ]
CL <-
  Clusters %>% group_by(.data$Cluster_Label) %>% top_n(5, .data$Occurrences)
CL
```


```{r}
# Keyword growth
topkw = KeywordGrowth(
  M,
  Tag = "ID",
  sep = ";",
  top = 15,
  cdf = TRUE
)
topkw$PRIMATES <- topkw$PRIMATES + topkw$PRIMATE
topkw$LEMURS <- topkw$LEMURS + topkw$LEMUR
topkw$`PROPITHECUS-VERREAUXI-COQUERELI` <-
  topkw$`PROPITHECUS-VERREAUXI-COQUERELI` + topkw$VERREAUXI
topkw <- select(topkw,-PRIMATE)
topkw <- select(topkw,-LEMUR)
topkw <- select(topkw,-`RING-TAILED LEMURS`)
topkw <- select(topkw,-VERREAUXI)
topkw <- select(topkw,-`GASTROINTESTINAL-TRACT`)
topkw = rename(topkw, P.V.COQUERELI = `PROPITHECUS-VERREAUXI-COQUERELI`)
topkw <- subset(topkw, Year >= 1990)
DF = reshape::melt(topkw, id = 'Year') # reshape original data structure

alltopkw = KeywordGrowth(
  allM,
  Tag = "ID",
  sep = ";",
  top = 15,
  cdf = TRUE
)
alltopkw$PRIMATES <- alltopkw$PRIMATES + alltopkw$PRIMATE
alltopkw <- select(alltopkw,-PRIMATE)
alltopkw = rename(alltopkw, P.D.EDWARDSI = `PROPITHECUS-DIADEMA-EDWARDSI`)
alltopkw <- subset(alltopkw, Year >= 1990)
allDF = reshape::melt(alltopkw, id = 'Year') # reshape original data structure

lemurtopkw = KeywordGrowth(
  lemurs,
  Tag = "ID",
  sep = ";",
  top = 15,
  cdf = TRUE
)
lemurtopkw$PRIMATES <- lemurtopkw$PRIMATES + lemurtopkw$PRIMATE
lemurtopkw$LEMURS <- lemurtopkw$LEMURS + lemurtopkw$LEMUR
lemurtopkw <- select(lemurtopkw,-PRIMATE)
lemurtopkw <- select(lemurtopkw,-LEMUR)
lemurtopkw <- select(lemurtopkw,-POPULATION)
lemurtopkw <- select(lemurtopkw,-CONSERVATION)
lemurtopkw <- select(lemurtopkw,-`MICROCEBUS-MURINUS`)
lemurtopkw <- subset(lemurtopkw, Year >= 1990)
lemurDF = reshape::melt(lemurtopkw, id = 'Year') # reshape original data structure

update_geom_defaults("text", list(size = 2.8))

ggplot(NULL, aes(Year, value, group = variable)) +
  geom_line(data = lemurDF, aes(color = "black")) +
  geom_line(data = allDF, aes(color = "blue")) +
  geom_line(data = DF, aes(color = "red")) +
  scale_shape_manual(values = 1:15) +
  scale_x_continuous(breaks = seq(1990, max(DF$Year), by = 10)) +
  scale_y_continuous() +
  labs(
    y = "Count",
    variable = "Keywords",
    colour = "Search Term:",
    title = "Keywords Usage Evolution Over Time"
  ) +
  scale_color_manual(
    labels = c("Lemur*", "Propithecus", "P. Coquereli"),
    values = c("black", "blue", "red")
  ) +
  facet_wrap(variable ~ ., ncol = 4, scales = "free") +
  geom_text(
    data = DF %>%
      arrange(desc(Year)) %>%
      group_by(variable) %>%
      slice(1),
    aes(label = value),
    position = position_nudge(2),
    hjust = 0.5,
    show.legend = FALSE
  ) +
  geom_text(
    data = allDF %>%
      arrange(desc(Year)) %>%
      group_by(variable) %>%
      slice(1),
    aes(label = value),
    position = position_nudge(2),
    hjust = 0.5,
    show.legend = FALSE
  ) +
  geom_text(
    data = lemurDF %>%
      arrange(desc(Year)) %>%
      group_by(variable) %>%
      slice(1),
    aes(label = value),
    position = position_nudge(2),
    hjust = 0.5,
    show.legend = FALSE
  ) +
  
  theme(
    panel.border = element_blank(),
    axis.title.x = element_blank(),
    axis.title.y = element_blank(),
    plot.title = element_text(face = "bold", size = 12),
    text = element_text(size = 10),
    panel.background = element_rect(fill = "white"),
    plot.background = element_rect(colour = NA),
    axis.text = element_text(size = 10),
    axis.line = element_line(colour = "black"),
    panel.grid.major = element_line(colour = "#f0f0f0"),
    panel.grid.minor = element_blank(),
    axis.ticks = element_line(colour = "black"),
    plot.margin = unit(c(5, 5, 5, 5), "mm"),
    strip.background = element_rect(colour = "#f0f0f0", fill = "#f0f0f0"),
    legend.position = "top",
    legend.justification = "right",
    legend.box.margin = margin(c(-27.5, 5, 5, 5)),
    legend.text =  element_text(size = 10),
    strip.text = element_text(face = "bold")
  )
```


```{r}
# Three field plots
threeFieldsPlot(M, fields = c("JI", "AU", "ID"), n = c(10, 10, 25))
```


```{r}
#The summary statistics of the network; The main indices of centrality and prestige of vertices.
NetMatrix <-
  biblioNetwork(M,
                analysis = "co-occurrences",
                network = "keywords",
                sep = ";")
netstat <- networkStat(NetMatrix)
summary(netstat, k = 10)
```


```{r}
#Thematic Evolution Analysis
nexus <-
  thematicEvolution(
    M,
    field = "ID",
    years = c(2000, 2010, 2020),
    n = 100,
    minFreq = 2
  )
plotThematicEvolution(nexus$Nodes, nexus$Edges)
```


```{r}
# Create a historical citation network
histResults <- histNetwork(M, sep = ";")
net <- histPlot(histResults,
                n = 17,
                size = 7,
                labelsize = 5)
```


```{r}
# biblioshiny()

```
