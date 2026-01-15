phttps://blog.runevision.com/2025/01/procedural-creature-progress-2021-2024.html

As part of the game I am working on, I wanted to create quick sketches of all the (possibly hundreds) of creatures in it. I might have mild aphantasia, or maybe I'm just unimaginative, but either way drawing even one creature often requires finding so many references that it takes longer to find the references than it does to actually make the drawing. This is tedious and frustrating.

So I decided to make a procedural generation tool.

I decided to limit the scope to tetrapods (i.e. four-limbed creatures) in order to make the scope a little more reasonable, and excluded certain outliers (see below). I wanted to create a system that would allow interpolating between any range of creatures smoothly simply by altering the input values. Output topology is isomorphic between creatures with the same limb plan. No consideration for movement is given; the point of this is to create a silhouette.

I have some personal interest in comparative anatomy, so I didn't need to do too much reading for this. Suffice it to say that all tetrapods possess a similar skeletal structure, with four limbs and an axial skeleton. 

 Some animals with iconic structures are consciously excluded from this generator. Their features may be modelled manually. These include:
- terrapins
- armadillo and pangolins
- moose
- peacocks
- flying squirrels
- gliding lizards

Snakes and caecilians are also excluded, because modelling a tube is trivial. (With apologies to snakes.)

In order to identify the range of creatures I would need to be able to create, I went and looked at the list of creatures I had written up and picked out the most "extreme" representatives to identify common features and help plot the range of parameters.

- Bat
- Falcon
- Feline
- Salamander
- Seal
- Mouse

(The skeleton is of a pigeon, but because they have the same major skeletal elements as a kestrel and I couldn't find a kestrel diagram specifically, I don't really care)

I've never created anything like this before, but intuitively, I think I will start with a skeleton, then add a bunch of blobs around each "bone". Each bone is not going to be 1:1 with an anatomical skeleton, because that would be ridiculously complex.

Parameters shared between all bones:
- Length ratio
- "Twist" or long-axis rotation
- Relative orientation (default 0, 0, 0)
Volume envelope parameters:
- Volume thickness ratio relative to default bone thickness
- Volume thickness taper (from proximal to distal)
- Proximal XY thickness ratio (0.5 to 2)
- Distal XY thickness ratio (0.5 to 2)
- Center thickness multiplier (Adds a "round shape" by adding thickness to the middle of a bone, -1 to 1)
- XY offset (Moves the resulting envelope, -1 to 1)

Global volume envelope parameters:
- Default bone thickness (ratio of absolute body size input)
- 

The only absolute input is the body size input, which is a single value in metres. It's sort of arbitrary but is about the same as the total length of the body. We also define a global volume skinning thing for the thickness of volumes (for when you want to create greyhounds  ↔ saint bernards)

\* You should make a diagram showing a variety of animal skeletons and each part highlighted in the same colour to show homology
\* You should also do a thing to add special structures like spines, antlers, horns

I LOVE HOMOLOGY!!

We start with the axial skeleton, which contains a head, neck, torso, and tail. The spine (i.e. from neck top to tail end) is a single path/curve object. The axial skeleton has global parameters and additionally:

Cervical curvature (-1 to 1)
Thoracic curvature (-1 to 1)
Lumbar curvature (-1 to 1)
Sacral curvature (-1 to 1)


Now the head is quite complex. We actually need to do a "pre-model" for this, which is then deformed according to a set of parameters. We have a neurocranium (or braincase), rostrum (snout/beak), mandible (lower jaw/beak), orbital regions, and ears.

Head:
Neurocranium:
Neurocranium length (0.5 - 2)
Neurocranium width (Proximal-distal length of braincase, 0.2 - 0.6)
Neurocranium height (0.5 - 2)
Dorsal slope (-1 to 1 where 1 is like. parrot skull and -1 is like. triangular like lizard)

\* Skull dimensions are relative to trunk dimensions

Rostrum:
Snout elongation ratio (0 - 2 relative to neurocranium length)
Snout width (0.2 - 1 relative to neurocraniuim width)
Snout height (0.2 - 1 relative to neurocranium height)
Snout attachment height (0.2 to 1 where )
Taper factor (0 - 1 where 0 means it does not taper at all and 1 means it becomes a point, scales both laterally and dorsoventrally)
Cross-section factor (0 - 2 where 2 means the beak is twice as wide as it is tall, 0.5 means the beak is half as wide as it is tall, 1 is a perfectly equal cross-section)
Curvature start (0 to 1, where 0 is at the very base and 1 is at the very distal tip)
Curvature (-1 to 1, sagittal aka up/down curve of rostrum)
Tip sharpness (0 - 1 where 0 means no sharpness and 1 means it becomes a point )

Mandible:
Length ratio: (0.5 - 1 relative to rostrum length)
Width (0.1 - 0.5 relative to snout width, controls lateral width)
Curvature start (0 to 1, where 0 is at the very base and 1 is at the very distal tip)
Curvature (-1 to 1, sagittal curvature of mandible)
Jaw gape (0 to 1 where 1 is like 90 degrees open and 0 is closed)

Orbital region:
Eye placement: forward-facing - lateral, and vertical position
Eye height (-1 to 1 where 1 is like a crocodile's eyes)
Eye size ratio: (0.05 to 0.3, relative to skull width)
Protrusion: (-1 to 1 where at -1 the eyes are very deep-set/sunken and at 1 they are bulging)

Ears:
Length
Width
Thickness
Rotation in pitch, yaw, tilt
Pointiness
Curvature (droopiness, 0 to 1)

The neck:
Number of vertebrae (1 - 25)\*
Neck elevation angle (from 0 to 90 degrees)
Neck flexibility (from 0 to 1)
Curvature type (-1 to 1)

Torso/trunk:

Thoracic region:
Rib expansion factor (i.e. barrel-chested vs narrow torso, basically rib flare, 0 to 1)
(Pectoral) girdle attachment scale (i.e. lateral width of shoulders, 0.5 to 2)


Abdominal region:
(Pelvic) girdle attachment scale (i.e. hip width, 0.5 to 2)


\* Fun fact! The majority of mammals have seven neck vertebrae. So you have the same number of neck bones as a giraffe. Unusually, sloths have 5 – 9, and manatees have 6. This doesn't hold true for aviforms, which have significant variation, between 9 and 25. Salamanders and most other amphibians only have 1, and I can't think of any exceptions.

Limb architecture. Tetrapod limbs are internally consistent so this is not as bad as it sounds. Each limb has a girdle, stylopod,  zeugopod, autopod. In humans,  this is:

(drawing of arms and legs)

Stylopod (upper limb):
Length ratio
Long-axis twist/rotation (angle of longitudinal twist)
Attachment height
Attachment rotation (lateral rotation)

Zeugopod:
Length ratio
Long-axis twist/rotation (angle of longitudinal twist)
Resting joint angle/Stylopod-zeugopod angle (0 - 150 degrees)

I think it's pretty neat that all tetrapod limbs are homologous structures. In particular I remember seals, which have flippers which are remarkably similar to human hands, with five digits just like us despite looking quite different externally.

For autopods, aka hands and feet, digits are split into two groups:
Length ratio (relative to zeugopod)
Membrane extent 0 - 1

Thumb/dewclaw:
Presence flag (y/n)
Length ratio
Thickness
Curvature
Orientation offset (-90 to 90) (rotation around limb axis/lateral rotation)
Has membrane? y/n

Remaining digits:
Count (0 - 4)
Digit length ratio
Digit length ratio taper (0 - 1) controls how distal digits shorten relative to proximal digits
Digit (lateral) spread
Digit fusion?
Digit curvature (from 0 to 1 where 1 is a semicircle? or in radians)
Digit orientation offset (controls rotation of digit around limb axis, -90 to 90)

(We control per-digit thingies for the primary and other digits)

\*The aye-aye has 6 digits, having evolved a pseudothumb, but the vast majority of tetrapods stop at having 5 digits

(Humans have webbing between our fingers too!)

Limb posture/stance:
Sprawl angle (where 0 is directly horizontal/flat and 90 is directly under body)
Elbow/knee orientation (0 to 1 where 0 is under body and 1 is out to side)
Duty factor bias 
Pelvic tilt (anterior/posterior, -45 to 45)

Tail:
Tail length
Tail base thickness
Taper rate
Lateral and vertical ratio


METABALLS! We can use metaballs. (Maybe make this a toggle-able option)

1. Create a bone envelope curve node tree or whatever for each bone
2. 