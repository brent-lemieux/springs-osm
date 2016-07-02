# springs-osm

# OpenStreetMap Data Case Study

## Map Area: Colorado Springs, CO, USA

Exploring and wrangling Open Street Map data for Colorado Springs

https://www.openstreetmap.org/relation/113141#map=11/38.8758/-104.7588

I chose Colorado Springs because Colorado is my favorite state to visit in both winter and summer.  I would like to live somewhere in the state at some point, so I am interested to see what the queries turn up.

## Problems encountered in the OSM data

I downloaded the XML file and ran it thru my cosprings.ipynb code.  I also converted the file to a database and ran queries.  The three main problems I found were:

- Inconsistent formatting of street types (ex. '1107 W 4th st', 'Rockhill road')

- Inconsistent formatting of phone numbers (ex. '+1-123-456-7890', '1234567890', '(123) 456-7890')

- Incomplete data points (ex. Not all nodes that mark businesses contain addresses)

- Incomplete data (ex. Only three nodes represented grocery stores.  This can't be correct in a city the size of Colorado Springs.)

### Street Type Problems

By running a python function to audit the street types, I was able to see which street addresses in the data set did not end in a word that met the expected standard.  This function returns a dictionary with the last word of a street address as the keys (if that word is not in 'expected'), and all of the street addresses that end with the non-conforming word as the values.

'''python
street_type_re = re.compile(r'\b\S+\.?$', re.IGNORECASE)

expected = ["Street", "Avenue", "Boulevard", "Drive", "Court", "Place", "Square", "Lane", "Road", 
            "Trail", "Parkway", "Commons","Circle","Highway","Parkway","Terrace","Trafficway","Way","Loop","Alley",
            "Point"]

def audit_street_type(street_types, street_name):
    m = street_type_re.search(street_name)
    if m:
        street_type = m.group()
        if street_type not in expected:
            street_types[street_type].add(street_name)

def is_street_name(elem):
    return (elem.attrib['k'] == "addr:street")

def audit_streets(osmfile):
    osm_file = open(osmfile, "r")
    street_types = defaultdict(set)
    for event, elem in ET.iterparse(osm_file, events=("start",)):
        if elem.tag == "node" or elem.tag == "way":
            for tag in elem.iter("tag"):
                if is_street_name(tag):
                    audit_street_type(street_types, tag.attrib['v'])
    osm_file.close()
    return street_types

audit_streets(OSM)
'''

After seeing how a lot of the street types were keyed in, I was able to create a dictionary where the incorrect formatting of the street types are the keys and the correctly formatted version are the values.  I then created the function update_street to replace the incorrect formatting with the correct formatting for street types found in 'mapping'.  This function does not fix all incorrectly formatted street types.  Not all street types are the last word of the street address.  For example, this function would not correct '1106 37th St. Suite 310' to '1106 37th Street Suite 310'.  To do this, I would need to create a function that iterates over each word looking for abbreviations.  However, update_street fixes the majority of the problems.

'''python
mapping = { "St": "Street","St.": "Street","Ct": "Court","Rd": "Road","Rd.": "Road","Ave": "Avenue","Ave.": "Avenue",
            "Blvd":"Boulevard","Blvd.":"Boulevard","Ct":"Court","Dr":"Drive","Dr.":"Drive","HWY":"Highway",
            "Hwy":"Highway","Ln":"Lane","Pkwy":"Parkway","RD":"Road","ST":"Street","STREET":"Street","Ter":"Terrace",
            "Trfy":"Trafficway","ave":"Avenue","circle":"Circle","ct":"Court","dr":"Drive","Pl":"Place","rd":"Road",
            "st":"Street","st.":"Street","street":"Street","ter":"Terrace","terrace":"Terrace"
            }

def update_street(name, mapping):
    s = street_type_re.search(name)
    if s:
        street_type = s.group()
        if (street_type not in expected) and (street_type in mapping.keys()):                                
            name = re.sub(street_type_re,mapping[street_type],name)                                
    return name
'''

