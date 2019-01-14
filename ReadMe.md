The map_server was a part of [Navigation Stack](http://wiki.ros.org/map_server?distro=melodic)

It provides the map_saver to save the map topic(nav_msgs/OccupancyGrid) to disk in `.pgm` + `.yaml` form.

While Cartographer's map topic is different from other's like GMapping, so we need to change some source code.

| point type | GMapping value | Cartographer value |
|:----:|:----:|:----:|
| free | 0 | 0-N |
| occupied | 100 | M-100 |
| unknown | -1 | -1 & N-M |

This package change the map_saver.cpp in function `void mapCallback(const nav_msgs::OccupancyGridConstPtr& map)` by

```cpp
for(unsigned int y = 0; y < map->info.height; y++) {
  for(unsigned int x = 0; x < map->info.width; x++) {
    unsigned int i = x + (map->info.height - y - 1) * map->info.width;
    if (map->data[i] >= 0 && map->data[i] < 40) { //occ [0,0.1)
      fputc(254, out);
    } else if (map->data[i] > +50) { //occ (0.65,1]
      fputc(000, out);
    } else { //occ [0.1,0.65]
      fputc(205, out);
    }
  }
}
```

You can change the value by your need.

You can add this package to your workspace, make it and run it by `rosrun map_server map_saver [-f mapname]`

If your workspace can not work with Cartographer, you can find `map_saver` in `your_ws/devel/lib/map_server` and run directly by `./map_saver [-f mapname]`. 

It should save the map in your current path or in the specific path if you point out by mapname
