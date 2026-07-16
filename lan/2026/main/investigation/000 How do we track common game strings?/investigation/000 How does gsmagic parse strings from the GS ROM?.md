---
context_type: investigation
status: todo
---

Parent: [lan/2026/main/investigation/000 How do we track common game strings?/000 How do we track common game strings?](../000%20How%20do%20we%20track%20common%20game%20strings%3F.md)

Spawned by: [lan/2026/main/investigation/000 How do we track common game strings?/000 How do we track common game strings?](../000%20How%20do%20we%20track%20common%20game%20strings%3F.md)

Spawned in: [<a name="spawn-invst-a14068" />^spawn-invst-a14068](../000%20How%20do%20we%20track%20common%20game%20strings%3F.md#spawn-invst-a14068)

# Objective

Produce a program as a resolution to parse strings from the ROM using the gsmagic's method.

# Resolution

TODO

# Journal

2026-07-08 Wk 28 Wed - 07:36 +03:00

````sh
# in /home/lan/Downloads/gsmagic_April_2020_-_height_tiles_saving_treasure_coords_and_class_type_patching/gsmagic/bin/Release
wine gsmagic.exe
````

It is able to load the map! There are a lot of other things too, items, artifacts, and text too. Although I am getting some exceptions and they're not loading.

````cs
// in /home/lan/Downloads/gsmagic_April_2020_-_height_tiles_saving_treasure_coords_and_class_type_patching/gsmagic/Form1.cs
			string[] greeting = {
				sayHi(), //Used for default message. (After the below messages are done.) And given the rng, it's not the same everytime!
				"Hello! Welcome to GSM! Nice to meet you!",
				"Hey there! What's your name again? Was it {0}? Welcome back!",
				"Hey {0}! I think I finally got your name now. What's up?",
				"Hey {0}! How's the hacking going?",

				"You sure do love to hack, don't you {0}?"

			};
````

Fun! I got the what's your name again one so I was surprised.

````cs
// in /home/lan/Downloads/gsmagic_April_2020_-_height_tiles_saving_treasure_coords_and_class_type_patching/gsmagic/Form1.cs
					switch (language) {
						case 'E':
						case 'I': filetable = 0x08320000; break;
						case 'J': filetable = 0x08317000; break;
						case 'D': filetable = 0x0831FE00; break;
						case 'F':
						case 'S': filetable = 0x08321800; break;
````

Tracks with [001 Golden Sun Master File Table Pointers](../../../wiki/001%20Wiki%20Entities/entry/001%20Golden%20Sun%20Master%20File%20Table%20Pointers.md).

````cs
// in /home/lan/Downloads/gsmagic_April_2020_-_height_tiles_saving_treasure_coords_and_class_type_patching/gsmagic/Form1.cs
                //Adjust Text Pointers - My editor does not edit text if you don't edit text, so adjust this.
                int asmptext = Bits.getInt32(rom, 0x385DC) - 0x8000000;
````

````cs
// in /home/lan/Downloads/gsmagic_April_2020_-_height_tiles_saving_treasure_coords_and_class_type_patching/gsmagic/Static/Do.cs
        public static void drawStr(int[] bmpdata, int x, int y, int stride, string str2draw, int color = 0xF8F8F8)
        {
            //str2draw = "EDITOR BY TEAWATER : BUG TESTED BY SALANEWT";//"0123456789:ABCDEFGHIJKLMNOPQRSTUVWXYZ";
            int[] hexList = {
                0x00075557,//0
                0x00072232,//1
                0x00071747,//2
````

Tracing from `Text Editor`,

* `gsmagic/Editors.cs > fn texteditor`
  * \|--- `gsmagic/Table Manager.cs > fn Table_Manager::doTableListbox`
    * note
      * prior calls `tm.doTableListbox(txt, Bits.numList(12461));`
      * `sl`: `public SearchList sl = new SearchList();`
  * $\to$ `gsmagic/Controls/SearchList.cs > fn SearchList::doTableListbox`
    * note
      * prior calls `sl.doTableListbox(pnl, txt, items);`
  * \|--- `gsmagic/Static/Bits.cs > fn Bits::NumList `
    * note
      * Just creates a list `[0, 1, 2, ...]` up to `range - 1`.

Seems empty.

Tracing from a function that seems obviously related to text parsing,

* `gsmagic/Static/Bits.cs > fn Bits::getTextShort`
  * note
    * Uses a `0xC300` offset.
