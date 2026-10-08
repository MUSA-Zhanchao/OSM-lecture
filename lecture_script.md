# Lecture Script — OpenStreetMap, Street Networks, and Pandana

MUSA-5500 · Week 8 · Zhanchao Yang
Deck: `OSM_Pandana_Lecture-revised-final.pptx` · 180 minutes

How to read this script:

- **Say** is what you say out loud. Treat it as a starting point and put it in your own words.
- **Do** is what you click or run.
- **If asked** gives short answers to questions students are likely to ask.
- The slide number is the page number printed on the slide. Times are minutes from the start of class.
- At the end there is a **Pandana background briefing** (Appendix A). Read it once tonight. It covers what you need to field questions without notes.

---

## Run of show

| Time | Block | Slides | Notebook |
|---|---|---|---|
| 0–26 | What OSM is and how it stores a city | 1–10 | Browser only |
| 26–79 | OSMnx: features, networks, a first route | 11–22 | `backup/01_OSM_Step_by_Step.ipynb` §1–§6 |
| 79–90 | Buffer: catch-up, questions, optional preview | — | Step-by-step §7 (optional) |
| 90–100 | Break | 24 | — |
| 100–117 | Why accessibility, what Pandana is, why use it | 25–29 | — |
| 117–155 | Tutorial Part 2, step by step | 30–35 | `week-8A-street-network.ipynb` Part 2 |
| 155–165 | Interpreting results and using Pandana well | 36–37 | — |
| 165–177 | Transit accessibility with r5r | 38–40 | — |
| 177–180 | Wrap-up and exit ticket | 41 | — |

## Before class checklist

1. **Environment.** `import pandana` and `from pandana.loaders import osm` should both work. The loader needs the separate `osmnet` package (`pip install osmnet`). If pandana won't install with pip, `conda install -c conda-forge pandana osmnet` is the most reliable fallback.
2. **Prerequisite cells for Part 2.** Part 2 of `week-8A` uses `center_city_outline`, and that variable is created in **Part 1 of the same notebook**, not in the step-by-step notebook. During the break, open `week-8A-street-network.ipynb` and run:
   - the first imports cell (`altair`, `geopandas`, `numpy`, `pandas`, `matplotlib`);
   - under **"Streets within a specific polygon"**: the `planning_districts = gpd.read_file(...)` cell, `central_district = planning_districts.query("dist_name == 'Central'")`, and `center_city_outline = central_district.squeeze().geometry`.
3. **Test the downloads once tonight.** Step 1 (`node_query`) and Step 2 (`pdna_network_from_bbox`) call the Overpass API. Step 2 took about 17 seconds plus preprocessing when the saved outputs were made. If Overpass is slow in class, show the saved outputs already in the notebook and keep talking. Every map is saved there.
4. **Offline fallback.** `backup/02_Pandana.ipynb` builds a Pandana network from the saved OSMnx graph (`data/walk.graphml`) and does not touch Overpass. It reads `data/...` paths, so start Jupyter from the repo root (or change the paths to `../data/...`).
5. **Known tutorial quirk.** `plot_walking_distance()` hardcodes `distance=1000` inside its call to `nearest_pois` and ignores its own `distance` argument. That's why the map colorbars stop at 1000 even though `set_pois` used 2000. You can use this in class (see slide 34).

---

# Part I — OpenStreetMap and OSMnx (0–90)

### Slide 1 — OpenStreetMap and street networks `0–2`

**Say:**
> Good afternoon. I'm Zhanchao. Today has two halves. In the first half we'll get to know a map dataset, OpenStreetMap, and write a few lines of Python to pull streets and places out of it. By the break we'll have a walking route from one point to a station. In the second half we go from one route to a whole city: for every street corner, how far is it to the nearest bar, school, or restaurant? That's where a library called Pandana comes in. At the very end I'll show what I use when the question involves transit.
>
> Every example today is in University City or Center City, so you'll recognize the places.

### Slide 2 — What is OpenStreetMap `2–5`

**Do:** Open the OSM link around Penn (in the slide notes) or just use the screenshot.

**Say:**
> Here's a map you've seen a hundred times. Point to three things for me: a street, a building, and a station.
>
> *(Take answers.)*
>
> Now the harder question: when you use a map app, what does it leave out? What do you wish it showed?

Let them answer. Don't explain graphs yet.

### Slide 3 — What is OpenStreetMap? `5–8`

**Say:**
> OpenStreetMap is not a picture. It's an editable geographic database. People add locations, shapes, and descriptions of places, and anyone can download and reuse those data. The map tiles you see on the website are just one way of drawing the database.
>
> Wikipedia is a fair comparison for how it's built, with many volunteers editing the same shared thing. The governance isn't identical, but the idea is close.

### Slide 4 — Who maintains the map? `8–11`

**Say:**
> Every object has an edit history. This is the 34th Street station. You can see who added it, who moved it, who fixed its tags. So the data are only as current as the last person who edited them.
>
> Question: a new café opens on Walnut Street tomorrow. Is it on the map tomorrow?
>
> *(Answer: only if someone adds it. That matters later when we count cafés.)*

### Slide 5 — How does OSM store a city? `11–13`

**Say:**
> OSM uses three building blocks. A **node** is a point with coordinates. A **way** is an ordered list of nodes, so it's a line, or an area if it closes and its tags say it's an area. A **relation** groups other objects together.
>
> That's all I want you to remember for now. The next slides show each one with a real object.

### Slide 6 — A station is a point with a description `13–16`

**Say:**
> The 34th Street station is one OSM node with several tags. Can you find the name? The network?
>
> One warning that matters later: an OSM *node* isn't the same as an *intersection* in a street network. This station is a node, but it isn't where two streets cross. Keep the two words separate in your head.

### Slide 7 — Locust Walk is an ordered line `16–19`

**Say:**
> Locust Walk is a way, an ordered line of nodes. Look at its tags: `highway=pedestrian`, `motor_vehicle=no`. Remember this one. Later we build a walking network and a driving network from the same area, and this tag is why they look different.

### Slide 8 — A relation groups existing objects `19–21`

**Say:**
> A relation records how objects belong together. A bus route, for example, is made of many street segments and many stops. We won't parse relations today. I just want you to know they exist.

### Slide 9 — Tags turn geometry into meaning `21–23`

**Say:**
> These are the four real tags on Locust Walk. Which tag best explains why you can walk here but usually can't drive?
>
> *(Answer: `highway` and `motor_vehicle`.)*
>
> Without tags, a line is just geometry. Tags turn it into a footpath, a road, or a rail line. Every query we write today is really a tag query.

### Slide 10 — What might the map miss? `23–26`

**Say:**
> A missing footpath. A closed entrance. A café that opened last week. OSM is good, but it's built by people, and coverage is uneven. When your analysis says "nobody can walk to X," one possible reason is "nobody mapped the path to X." Keep that in mind all afternoon.

### Slide 11 — How can we use these data in Python? `26–29`

**Say:**
> OSM supplies the data. OSMnx is a Python package that downloads it and turns it into things we can analyze: GeoDataFrames and street graphs. First we'll find a place and pull a few features.

### Slide 12 — Our first notebook cells `29–30`

**Do:** Open `backup/01_OSM_Step_by_Step.ipynb`, run §1.

**Say:**
> One cell at a time: run it, look at the output, then move on. The cache setting just reuses downloads we've already made, so we're not waiting on the server.

### Slide 13 — First, find Philadelphia `30–35`

**Do:** Run §2: `ox.geocode_to_gdf(...)`, then `philly.plot()`.

**Say:**
> Before we plot it, look at what came back. Is it a picture of a map, or data we can keep analyzing?
>
> *(Answer: a GeoDataFrame.)* It has a geometry column, a CRS, everything you already know from GeoPandas.

**If the download fails:** open `backup/01_OSM_Offline_Backup.ipynb` at the same section.

### Slide 14 — A buffer needs meaningful units `35–38`

**Do:** Run `philly.crs`, then `to_crs("EPSG:32618")`.

**Say:**
> What CRS is this? EPSG:4326, longitude and latitude in degrees. If I want an 800-meter circle, degrees don't work. One degree of longitude isn't the same distance as one degree of latitude here. So we project to UTM zone 18N, where the units are meters. That's all the projection theory we need today.

### Slide 15 — What is around 34th Street? `38–44`

**Do:** Ask for guesses first, then run §3.

**Say:**
> Before I run this: how many cafés are within one kilometer of the 34th Street station? Make a guess.
>
> *(Run.)*
>
> Two details. The center is written (latitude, longitude): y first, then x. And the tag is a dictionary, `amenity: cafe`, so we're asking OSM for anything with that tag.

### Slide 16 — Inspect before plotting `44–47`

**Say:**
> Let's read one row. Each row is one OSM feature. Some have a name, some don't. Some are points, some are building outlines. So 25 café features doesn't mean 25 verified businesses. That's a data question, not a Python question, and it comes back in the second half.

### Slide 17 — Your turn: find restaurants `47–50`

**Say:**
> Which one word would you change to find restaurants?
>
> *(Answer: `cafe` → `restaurant`.)*
>
> If the network's behaving, go ahead and run it. Otherwise just write the change and predict what you'd see.

### Slide 18 — Now we want connections `50–56`

**Do:** Run §4: `ox.graph_from_point(center, dist=2500, network_type="walk")`, then `ox.plot_graph`.

**Say:**
> Points tell us *where* things are. They don't tell us *how to get there*. For that we need connections: a street network.
>
> We download a larger area than we'll analyze, 2.5 km, so routes near the edge have room to go around things. Don't worry about the word "MultiDiGraph" yet. Just look at the shape.

### Slide 19 — Can walking and driving use the same links? `56–61`

**Do:** Show the driving graph first. Ask what walking will add, then show walking.

**Say:**
> Same center, same 600 meters. The only difference is `network_type`. What does the walking network have that the driving one doesn't?
>
> *(Paths, campus walkways, Locust Walk.)* That's the tag from slide 7 at work.

### Slide 20 — The dots and lines have names `61–66`

**Do:** Run `type(G)` and `len(G.nodes), len(G.edges)`.

**Say:**
> On the left are all the OSM shape points. On the right is the simplified graph. OSMnx keeps a node only where streets actually connect or end. A bend in the road doesn't have to be an intersection. The curve is still kept as the edge's geometry.
>
> It's a *directed* graph, because a one-way street only goes one way, and it allows *parallel* edges, two different streets between the same pair of corners. That's what MultiDiGraph means.

### Slide 21 — The graph becomes two familiar tables `66–71`

**Do:** Run §5: `project_graph`, `graph_to_gdfs`, then look at `nodes` and `edges`.

**Say:**
> The graph can become two tables you already know how to handle. Nodes have locations. Edges have connections, a `length` in meters, a `highway` tag, and a geometry. Edges are indexed by `(u, v, key)`: from-node, to-node, and which parallel edge.
>
> **Remember this slide.** In the second half, Pandana takes exactly these two tables, a node table and an edge table.

### Slide 22 — A first walking route `71–79`

**Do:** Run §6: `origin`, `station_node`, then `shortest_path`, `route[:5]`, `plot_graph_route`.

**Say:**
> `nearest_nodes` snaps a coordinate to the closest graph node. Careful: here X is longitude and Y is latitude, the opposite of the `center` tuple. Coordinate order will catch you all afternoon.
>
> The route is just a list of node IDs. Then we draw it.
>
> That's one origin, one destination, one route. Hold on to that thought.

### Buffer `79–90`

Use this time to catch up whoever is behind and take questions. If the room is ahead, run step-by-step §7, the 800 m circle around the station, as a preview: "Is everything inside this circle actually an 800 m walk away? We'll answer that after the break."

### Slide 24 — Break `90–100`

**Say:**
> Ten minutes. Leave your notebook running. When we come back we go from one route to the whole city.

**Do (during the break):** In `week-8A-street-network.ipynb`, run the prerequisite cells from the checklist so that `center_city_outline` exists.

---

# Part II — From routing to accessibility with Pandana (100–180)

### Slide 25 — One route is easy. What about the whole city? `100–102`

**Say:**
> Before the break we had one origin, one destination, one route. Now the question changes. Planners don't usually ask "how do I get from here to there?" They ask "from *every* place in this neighborhood, how far is the nearest grocery store? Which blocks are more than ten minutes from a school?" That's an accessibility question, and it has thousands of origins.
>
> Here's the plan for the next 80 minutes: fifteen minutes on what Pandana is and why we use it, then we work through the tutorial together, and we finish with transit.

### Slide 26 — Proximity and accessibility `102–106`

**Say:**
> Name a place near campus that's close but annoying to reach. Across the river, behind the rail yard, on the other side of the Schuylkill Expressway.
>
> *(Take one or two.)*
>
> That's the difference. **Proximity** is how close something is in a straight line. **Accessibility** is whether you can actually get there within some cost: a distance, or a time. Same destination, different distance, and a different planning question. Everything we compute from here on is the right-hand picture: distance *along the network*.

### Slide 27 — What is Pandana? `106–111`

**Say:**
> Pandana stands for **Pandas Network Analysis**. It's an open-source Python library from UDST, the Urban Data Science Toolkit, the same group behind UrbanSim, a land-use simulation model. It was built for one job: accessibility queries on big street networks, fast.
>
> Three things to know:
>
> 1. **The input is just two tables.** A node table with an ID, x, and y, and an edge table with from, to, and a distance. The same two tables OSMnx gave us before the break.
> 2. **It preprocesses the network once.** When you build a Pandana network it runs a step called *contraction hierarchies*, written in C++. You'll see it print "Generating contraction hierarchies" in the notebook. That step takes a few seconds, and afterwards every query is very fast.
> 3. **Every answer is a pandas DataFrame with one row per network node.** So you never loop over origins yourself. Every street corner gets an answer at once.
>
> There are two main kinds of query. `nearest_pois` answers "how far is the 1st, 2nd, nth nearest place?", and that's what the tutorial uses. `aggregate` answers "what's the total of something within this distance?", like how many jobs are within 800 meters. We'll come back to that at the end.

**If asked "What are contraction hierarchies?":**
> Before any query, the algorithm ranks nodes by importance and adds shortcut edges, so a long trip can skip over many small streets. A search then only has to climb toward important nodes from both ends instead of spreading out in every direction. You pay once at setup, and every query is cheap. Google-Maps-style routers use the same family of ideas.

### Slide 28 — Why Pandana? `111–114`

**Say:**
> Here are real numbers from the notebook. The Center City walking network has **30,166 nodes**. We'll ask, for every one of them, how far it is to the 10 nearest bars.
>
> With what we did before the break, `ox.shortest_path`, that's one search per origin, so a loop of thirty thousand searches. You can do it, and for one route OSMnx is the right tool. But this question has thousands of origins, and you'll want to rerun it for schools, restaurants, and different distances.
>
> With Pandana it's **one function call**, and every node gets an answer in a few seconds. That's the reason to use it. Not that it can route, since OSMnx can too, but that it can ask the same question from the whole network at once, over and over.

### Slide 29 — Two tools in the same analysis `114–117`

**Say:**
> So it isn't OSMnx *or* Pandana. They split the work. OSMnx is great for getting streets and features, building and checking the graph, and getting a single route with its real geometry. Pandana attaches places to the network and asks every node at once.
>
> One practical note: in today's tutorial Pandana downloads its own network with a loader called `osmnet`, so it's a separate download from the OSMnx graph. If you've already cleaned a network in OSMnx for a project, you can hand OSMnx's node and edge tables straight to Pandana instead. The backup notebook shows how.

### Slide 30 — The tutorial in four steps `117–119`

**Do:** Switch to `week-8A-street-network.ipynb`, scroll to **"Part 2: Pandana"**.

**Say:**
> This is the map for the rest of the tutorial. Four steps, and every slide from here has the step number in its title.
>
> 1. **Get amenities.** Pull every OSM point tagged `amenity` in Center City.
> 2. **Build the network.** Download the walking network and preprocess it.
> 3. **Attach the places.** Tell the network where the restaurants, bars, schools, and car-share spots are.
> 4. **Query and map.** Ask every node how far the nearest ones are, then map it.
>
> The numbers on the cards are from the real run: 3,396 amenities, 30,166 nodes, 44,292 edges.

### Slide 31 — Steps 1–2: amenities and a walking network `119–127`

**Do (live):**
1. Run the import cell. It also patches the User-Agent.
2. Run `boundary = center_city_outline.bounds` and the unpacking line.
3. Run `osm.node_query(...)`, then `poi_df.head()`, `len(poi_df)`, and the Altair bar chart.
4. Run `osm.pdna_network_from_bbox(...)` and talk while it downloads.

**Say (import cell):**
> The import cell has a small patch. The Overpass server now rejects requests that don't say who they're from, and `osmnet` doesn't send that header. These lines just add one. You don't need to understand it. It only makes the download work.

**Say (bounding box):**
> This is the **number one bug** in this tutorial. Shapely's `.bounds` gives you (min x, min y, max x, max y), which is longitude first. Pandana's loaders want **latitude first**: (lat_min, lng_min, lat_max, lng_max). So we unpack the box into named variables and pass them in the order the loader expects.

**Say (Step 1):**
> `node_query` asks OSM for every *node* with an amenity tag. Note: nodes only. A restaurant drawn as a building outline won't show up here. That's different from OSMnx's `features_from_...`, which also returns polygons.
>
> *(After the bar chart.)* What's the most common amenity in Center City?

**Say (Step 2, while it runs):**
> This one takes about twenty seconds. It downloads every walkable way in the box, builds node and edge tables, and then you'll see it print "Generating contraction hierarchies." That's the preprocessing I mentioned. It's done once, and everything after this is quick.
>
> *(When finished.)* 30,166 nodes and 44,292 edges. Edge lengths are in meters. Node coordinates are longitude and latitude.

**If Overpass hangs:** "The server's busy, so let's read the saved output." Scroll through the saved outputs and continue. Or switch to `backup/02_Pandana.ipynb`, which builds the network from the saved OSMnx graph.

### Slide 32 — Step 3: tell the network what to look for `127–135`

**Do (live):** Run the `max_distance` / `num_pois` / `for amenity in AMENITIES` cell.

**Say:**
> `set_pois` does three things.
>
> 1. **It registers a category**, a name we'll use later, like `"bar"`.
> 2. **It snaps each place to its nearest network node.** In the picture, the orange dots are the places and the black dots are the nodes they snap to. From now on Pandana only knows the node, not the exact coordinate.
> 3. **It sets a search limit.** Here that's 2,000 meters and at most 10 places per node. Pandana won't look further than this, and you can't ask for more later.
>
> Look at the order of arguments: category, max distance, max items, then **x, then y**. x is longitude, so `lon` goes first. This is the opposite of the bounding box. That's exactly why I keep warning you about coordinate order.
>
> And note the comment in the notebook: only these four categories are registered. To look at cafés, add `"cafe"` to the list and rerun this cell.

**If asked "Why a max distance at all?":**
> It's what makes the query fast. Pandana precomputes, for each node, which places are within that distance. A bigger limit means more work and more memory. Pick the largest distance you'll actually ask about.

### Slide 33 — Step 4: ask every node at once `135–143`

**Do (live):** Run `access = net.nearest_pois(distance=2000, category="bar", num_pois=num_pois)`, then `access.tail(n=50)`.

**Say:**
> One call: for every node, how far along the network to the 1st, 2nd, through 10th nearest bar.
>
> Read the output. The **index is the node ID**. The **columns are named 1 through 10**: column 1 is the nearest, column 2 the second nearest, and so on. These two rows are copied from the notebook. Node A, in Old City, has three bars within about 500 meters. Node B, down by the river, has its nearest bar at 1,745 meters, and then **2,000, 2,000**.
>
> That 2,000 doesn't mean there's a bar exactly two kilometers away. It means **none was found within the 2,000-meter limit**. Pandana fills those cells with the limit. If you average this column or map it without thinking, you'll treat "none" as "two kilometers." Scroll through `access.tail(50)` and find some 2,000s yourself.
>
> And note what this *isn't*: it isn't a route. No geometry, just distances. If you want to draw the path, go back to OSMnx.

### Slide 34 — Step 4: merge, then map `143–150`

**Do (live):** Run `net.nodes_df.head()`, `access.head()`, the `pd.merge` cell, then define `plot_walking_distance` and run the nearest-bar plot.

**Say:**
> To map it we need coordinates. `net.nodes_df` has x and y for every node. `access` has the distances. They share the same index, the node ID, so we merge on the index. Then we turn it into a GeoDataFrame and color each node by column 1.
>
> *(Map appears.)* Each dot is a street node. Yellow-green means a short walk to the nearest bar, purple means a long one. The red stars are the bars.

**Teachable moment (the tutorial quirk):**
> Look at the colorbar. It stops at 1,000, but we set 2,000. Why? Look inside `plot_walking_distance`: it calls `nearest_pois(distance=1000, ...)`, hardcoded, so the `distance` argument of the function is never used. So every purple dot here means "**nothing within 1,000 meters**," not "exactly 1,000." This is the same lesson as the last slide: the cap is not a measurement.
>
> *(Optional fix:)* change `distance=1000` to `distance=distance` inside the function.

### Slide 35 — Nearest vs. 3rd nearest `150–155`

**Do (live):** Run the bar plots for `n=1` and `n=3`. Then let students run school, restaurant, and car-share on their own.

**Say:**
> Same query, different column. On the left, the nearest bar. On the right, the third nearest.
>
> Find a place that's green on the left and purple on the right. People there have *one* bar nearby, so they have access but no choice. In Old City or around Rittenhouse it's green on both, so they have choices.
>
> **The nearest tells you about access. The nth nearest tells you about choice.** That's why we asked for 10 and not 1.
>
> Now try it yourself: schools, restaurants with `n=10`, car sharing. What surprises you?

### Slide 36 — From nearest POI to accessibility `155–159`

**Say:**
> Pandana gives us a *distance*. To turn that into an *access* answer you need a threshold, a budget. Say 800 meters, roughly a ten-minute walk. If the network distance is under the budget, the place is accessible. Otherwise it isn't.
>
> The same output answers different planning questions. Nearest grocery store is column 1. Parks within 800 meters means counting how many columns are under 800, or using `aggregate`. Gaps are the nodes where column 1 is over the budget.
>
> The software gives you the distance. **The threshold is your judgment**, and you should be able to defend why you chose it.

### Slide 37 — Using Pandana well `159–165`

**Say:**
> Six habits that will save you on a project. Every one of them showed up in what we just ran.
>
> 1. **Match the limit to your question.** `nearest_pois` can't look past the `maxdist` or the number of items you gave `set_pois`. Set them for the biggest question you'll ask.
> 2. **Don't treat the limit as a value.** A value equal to the limit means nothing was found. Turn it into NaN, or label it "more than 2 km," before you map it or average it.
> 3. **Pad the bounding box.** Bars just outside the box don't exist as far as the network knows, so nodes near the edge look worse than they really are. Download a bigger box, analyze, then clip back to your study area.
> 4. **Check what counts as a place.** Look at `poi_df` before trusting a category. Look for duplicates, missing names, and places mapped as buildings that `node_query` misses.
> 5. **Watch coordinate order.** The loaders take latitude first. `set_pois` and `get_node_ids` take x = longitude first. Swapping them doesn't raise an error. The points just snap to the wrong nodes.
> 6. **Count, not only nearest.** For "how many jobs within 800 m," use `net.set(node_ids, jobs)`, then `net.aggregate(800, type="sum", decay="flat")`. `decay="flat"` means a plain count. The default is linear decay, which weights closer things more.

### Slide 38 — Pandana or r5r? `165–169`

**Say:**
> Last question: what if the destination is a bus or train ride away?
>
> In Pandana every edge has **one fixed cost**, meters or minutes. That's fine for walking and driving. But transit doesn't work that way. If you leave at 8:00 you might catch a train in two minutes. At 11 p.m. you might wait twenty. Waiting and transfers depend on the **timetable**, and Pandana has no timetable.
>
> For that I use **r5r**. It's an R package that routes on streets *and* transit schedules. If you prefer Python, **r5py** wraps the same engine.
>
> The table sums it up. Pandana: fast, citywide, walk or drive, "how far?" r5r: transit-aware, "how long, if I leave at 8:00?"

### Slide 39 — r5r: transit accessibility in R `169–173`

**Say:**
> r5r takes two kinds of input: a street network as an OSM `.pbf` file, and transit schedules as GTFS zip files. SEPTA publishes GTFS for bus and rail. You can add an elevation raster for walking and biking on hills if you want.
>
> `build_network()` builds a multimodal network using R5, the routing engine developed by Conveyal. It runs in Java, so you need JDK 21 installed. The first time, r5r downloads the R5 engine for you.
>
> Then there are four functions I use most:
> - `travel_time_matrix()`: minutes between every origin and destination.
> - `accessibility()`: how many opportunities, like jobs, are reachable within a cutoff.
> - `detailed_itineraries()`: the actual trips, with walk, wait, ride, and transfer legs as lines you can map.
> - `isochrone()`: the area you can reach within N minutes.

*(If you have a figure from your own r5r project, show it here. One real example from you is worth more than this slide.)*

### Slide 40 — r5r in practice `173–177`

**Say:**
> I won't run this. Just read it with me and compare it to Pandana's four steps.
>
> `build_network` is "build the network." The `jobs` column on the destinations is the opportunities, like `set_pois`. `accessibility()` is the query. `cutoffs` is the threshold. It's the same logic.
>
> Three things are new because of transit:
> - **`departure_datetime`.** Transit access changes by the minute, so pick a realistic weekday morning. This date has to fall inside the GTFS service calendar. If it doesn't, r5r finds no transit trips and you silently get walk-only results. That's the most common r5r mistake.
> - **`time_window`.** Instead of one departure, r5r runs one per minute across, here, 60 minutes, because your wait depends on when you show up.
> - **`percentiles`.** Across those departures, the 25th percentile is a lucky trip, the 50th typical, and the 75th unlucky.
>
> The `-Xmx4G` line gives Java four gigabytes of memory. For one city that's usually enough.

### Slide 41 — OSMnx represents movement; Pandana measures access `177–180`

**Say:**
> Let's put the whole afternoon together. OSM is the data. OSMnx turns it into a graph you can inspect and route on. Pandana turns that network into answers for every node at once: nearest places, or totals within a distance. A threshold turns those answers into accessibility. And when the trip involves transit, add GTFS and use r5r.
>
> The most important line on this slide: **the software answers the network query. You decide what counts as an opportunity, what the cost is, and what threshold is acceptable.**
>
> Exit ticket: write down one finding from your maps, one assumption we made, and one thing you'd check next. For homework, the at-home exercise at the bottom of the notebook: pick a neighborhood and an amenity and explore.

---

# Appendix A — Pandana background briefing (read tonight)

## A1. The mental model in one paragraph

Pandana is a network that already knows where the places are. You build it once from a node table and an edge table. You register places on it, and each place gets snapped to its nearest node. Then you ask a question *from every node at once*, and get back a DataFrame indexed by node ID. It never returns routes, only numbers per node. The heavy lifting is a one-time preprocessing step (contraction hierarchies) plus a per-distance precomputation, both in C++, so repeated questions are cheap.

## A2. Where it comes from

- Pandana = **Pan**das **N**etwork **A**nalysis. Maintained by UDST (Urban Data Science Toolkit), the open-source group behind UrbanSim, a land-use simulation model.
- Original author: Fletcher Foti, in Paul Waddell's group at UC Berkeley. Background paper: Foti, Waddell & Luxen (2012), *A Generalized Computational Framework for Accessibility: From the Pedestrian to the Metropolitan Scale*.
- Its contraction-hierarchies code descends from the OSRM routing project (Dennis Luxen).
- Docs: https://udst.github.io/pandana/ · Code: https://github.com/UDST/pandana

## A3. The API you actually need

```python
import pandana as pdna
from pandana.loaders import osm          # needs the separate osmnet package

# --- Build a network: option 1, Pandana's own loader (today's tutorial)
net = osm.pdna_network_from_bbox(lat_min, lng_min, lat_max, lng_max,
                                 network_type="walk")    # or "drive"
# edges weighted by "distance" in meters; nodes in lon/lat

# --- Build a network: option 2, from OSMnx tables (backup/02_Pandana.ipynb)
nodes, edges = ox.graph_to_gdfs(G_projected)
edges = edges.reset_index().sort_values("length").drop_duplicates(["u", "v"])
net = pdna.Network(nodes.x, nodes.y, edges.u, edges.v,
                   edges[["length"]], twoway=False)    # keep OSMnx's directed edges

# --- What's inside
net.nodes_df        # index = node id, columns x, y
net.edges_df        # from, to, weight column(s)

# --- Places (POIs) and nearest queries
net.set_pois(category, maxdist, maxitems, x_col, y_col)  # x = lon, y = lat
access = net.nearest_pois(distance, category, num_pois=1)
#   -> DataFrame, index = node id, columns 1..num_pois
#   -> cells with nothing found are filled with `distance` (set max_distance= to change the fill)
#   -> include_poi_ids=True also returns which POI is 1st, 2nd, ...

# --- Snapping points yourself
node_ids = net.get_node_ids(x_col, y_col)     # nearest node for each point

# --- Aggregation ("how much is within d?")
net.set(node_ids, variable=jobs, name="jobs") # variable=None counts locations
net.precompute(800)                           # optional, speeds up repeated aggregate calls
net.aggregate(800, type="sum", decay="flat", name="jobs")
#   type:  sum, count, mean/ave, min, max, median, 25pct, 75pct, std
#   decay: flat (no weighting), linear (default), exponential  (decay only affects sum and mean)

# --- Diagnostics
net.low_connectivity_nodes(impedance=1000, count=10)  # nodes stuck in tiny islands
net.shortest_path_lengths(origins, destinations)       # vectorized OD distances
```

## A4. How the tutorial's numbers fit together

| Thing | Value in the tutorial | Where it is set |
|---|---|---|
| Study area | Center City planning district bounding box | `center_city_outline.bounds` |
| Amenity points | 3,396 | Step 1, `len(poi_df)` |
| Network | 30,166 nodes, 44,292 edges | Step 2 printout |
| Search limit | 2,000 m, 10 items per node | Step 3, `max_distance`, `num_pois` |
| Query | `distance=2000`, `num_pois=10` | Step 4.1 |
| Plot function query | `distance=1000` (hardcoded) | inside `plot_walking_distance` |

## A5. Likely questions and short answers

**"Is Pandana faster than NetworkX / OSMnx?"**
For one route, no real difference, and OSMnx gives you geometry. For the same question from thousands of origins, yes, by a lot. That's what the preprocessing pays for.

**"Is the distance straight-line or along streets?"**
Along streets. It's the shortest network distance in meters. Only the snap from a place to its nearest node is straight-line.

**"Why do some cells say exactly 2000?"**
Nothing was found within the search limit. It's the limit, not a distance. Mask it before you analyze.

**"Can I use time instead of distance?"**
Yes. Give the edges a time column (length ÷ speed) and use that as the impedance. Your `distance` arguments are then in minutes. It's still one fixed cost per edge, though, so it can't model waiting for transit.

**"Does it handle one-way streets?"**
Through `twoway`. The walk loader treats edges as two-way. When you build from OSMnx tables, `twoway=False` keeps OSMnx's directed edges.

**"Why can't I find café data after Step 3?"**
Only the four categories in `AMENITIES` were registered. Add `"cafe"` and rerun the `set_pois` cell.

**"Why is a place missing that I know exists?"**
Three possible reasons: it isn't in OSM, it's mapped as a building polygon (`node_query` only returns nodes), or it's outside the bounding box.

**"What's the difference between `nearest_pois` and `aggregate`?"**
`nearest_pois` gives distances to the nearest k places, which tells you about access and choice. `aggregate` gives a total or summary of a variable within a distance, like jobs or population, which tells you about opportunity volume.

**"Can Pandana do transit?"**
Not with timetables. You can fake a "transit" network with fixed edge times, but waiting and transfers won't be realistic. Use r5r or r5py.

**"Installation fails."**
Use `conda install -c conda-forge pandana osmnet`. The OSM loader needs `osmnet` separately.

## A6. Snapping, in one more sentence

Pandana snaps each place to its nearest node in a straight line, and by default every place gets snapped, however far away it is. The network distance is then measured from that node, not from the place itself. A place that's far from any street (a park centroid, a station in the middle of a rail yard) can snap somewhere misleading, so check a few on a map. Both `set_pois` and `get_node_ids` accept `mapping_distance=` to drop places that are too far from any node.

---

# Appendix B — r5r quick reference

```r
options(java.parameters = "-Xmx4G")      # memory for Java; set before library(r5r)
library(r5r)

net <- build_network(data_path = "data/philly")   # folder: *.osm.pbf + one or more GTFS *.zip

ttm <- travel_time_matrix(net, origins = pts, destinations = pts,
                          mode = c("WALK", "TRANSIT"),
                          departure_datetime = as.POSIXct("2026-10-13 08:00"),
                          time_window = 60, max_trip_duration = 60)

acc <- accessibility(net, origins = pts, destinations = pts,
                     opportunities_colnames = "jobs",
                     mode = c("WALK", "TRANSIT"),
                     departure_datetime = as.POSIXct("2026-10-13 08:00"),
                     time_window = 60, percentiles = c(25, 50, 75),
                     decay_function = "step", cutoffs = c(30, 45))
```

- Points: a data.frame with `id`, `lat`, `lon`, or a WGS84 `sf` POINT object.
- `decay_function`: `step` (count within cutoff), `exponential`, `fixed_exponential`, `linear`, `logistic`.
- Other functions: `detailed_itineraries()`, `expanded_travel_time_matrix()` (splits time into access, waiting, in-vehicle, and transfer), `isochrone()`, `pareto_frontier()` (time vs. fare).
- Common mistakes: the departure date falls outside the GTFS calendar, the Java version is wrong, or there isn't enough memory.
- Developed by IPEA (Brazil). Docs: https://ipeagit.github.io/r5r/ · Python: https://r5py.readthedocs.io/
