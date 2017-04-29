---
title: Setting HUGE areas of blocks in PocketMine plugins
layout: post
permalink: setting-huge-areas-of-blocks
published: true
---
Settting huge amounts of blocks inside a PocketMine plugin can be a tricky matter. It can block the main thread and cause major packet loss. Sadly, most plugins ignore this and do it anyway. **There is a better way!**

While developing *MineReset*, I was confronted with lagg issues regarding mine reset times. I was asked by many server owners to figure out some way to speed them up and avoid player loss. 

### The plain ol'block set
Let's start with the basics, this is how we set a rectangular area of blocks. While this block is executing, the server can't perform other tasks like responding to players.

```php
for($x = $xMin; $x <= $xMax; $x++){
	for($y = $yMin; $y <= $yMax; $y++){
    	for($z = $zMin; $z <= $zMax; $z++){
        	$level->setBlock(new Vector3($x, $y, $z), Block::get(0));
        }
    }
}
```

### The tick separated set
We can be craftier though. What if we only set small chunks of blocks every tick? Then the server could respond to player events in between the block sets. This can be fairly easily accomplished using a `PluginTask`. We set blocks in the same loop structure as before, but when we reach a threshold, we break and continue on the next tick.


```php
class BlockSetTask extends PluginTask{
	public function __construct(PluginBase $plugin, Vector3 $min, Vector3 $max, Level $level, $threshold = 100){
    	parent::__construct($plugin);
        $this->min = $min;
        $this->max = $max;
        $this->level = $level;
        $this->threshold = $threshold;
        $this->onRun(0);
    }
    public function onRun($tick){
    	$set = 0;
    	for($x = $this->min->x; $x <= $this->max->x; $x++){
        	for($y = $this->min->y; $y <= $this->max->y; $y++){
            	for($z = $this->min->z; $z <= $this->max->z; $z++){
                	$this->level->setBlock(new Vector3($x, $y, $z), Block::get(0));
                    $set++;
                    if($set === $this->threshold){
                    	$this->getOwner()->getServer()->getScheduler()->scheduleDelayedTask($this, 1);
                        return;
                    }
                	$min->z++;
                }
            	$min->y++;
            }
        	$min->x++;
        }
    }
}
```

### The async set
An asyncronous set method is by far the most complicated. It requires you to detect which chunks are needed, serialize those chunks, copy them to another thread, unserialize and process them and then inject them into the world again. This makes sense for *MineReset* because *MineReset* has another order of complexity in calculating ratio values. However, for a standard plugin, this may be overkill. The following is modified code from *MineReset*, the full code is [on GitHub](https://github.com/Falkirks/MineReset).

#### `Mine`

```php
class Mine{
    public $a, $b, $lev, $data;
    /** @var MineReset  */
    private $base;
    public function __construct(MineReset $base, Vector3 $a, Vector3 $b, Level $level, array $data = []){
        $this->a = $a;
        $this->b = $b;
        $this->base = $base;
        $this->data = $data;
        $this->level = $level;
    }
    public function isMineSet(){
        return (count($this->data) != 0);
    }
    public function setData(array $arr){
        $this->data = $arr;
    }
    public function getA(){
        return $this->a;
    }
    public function getB(){
        return $this->b;
    }
    public function getLevel(){
        return $this->level;
    }
    public function getData(){
        return $this->data;
    }
    public function resetMine(){
        $chunks = [];
        for ($x = $this->getA()->getX(); $x-16 <= $this->getB()->getX(); $x += 16){
            for ($z = $this->getA()->getZ(); $z-16 <= $this->getB()->getZ(); $z += 16) {
                //$this->getLevel()->getServer()->getLogger()->info(Level::chunkHash($x >> 4, $z >> 4));
                $chunk = $this->level->getChunk($x >> 4, $z >> 4, true);
                $chunkClass = get_class($chunk);
                $chunks[Level::chunkHash($x >> 4, $z >> 4)] = $chunk->toBinary();
            }
        }
        $resetTask = new MineResetTask($chunks, $this->a, $this->b, $this->data, $this->getLevel()->getId(), $chunkClass);
        $this->base->scheduleReset($resetTask);
    }
}
```

#### `MineResetTask`
```php
class MineResetTask extends AsyncTask{
    private $chunks;
    private $a;
    private $b;
    private $ratioData;
    private $levelId;
    public function __construct(array $chunks, Vector3 $a, Vector3 $b, array $data, $levelId, $chunkClass){
        $this->chunks = serialize($chunks);
        $this->a = $a;
        $this->b = $b;
        $this->ratioData = serialize($data);
        $this->levelId = $levelId;
        $this->chunkClass = $chunkClass;
    }
    /**
     * Actions to execute when run
     *
     * @return void
     */
    public function onRun(){
        $chunkClass = $this->chunkClass;
        /** @var  Chunk[] $chunks */
        $chunks = unserialize($this->chunks);
        foreach($chunks as $hash => $binary){
            $chunks[$hash] = $chunkClass::fromBinary($binary);
        }
        $sum = [];
        $id = array_keys(unserialize($this->ratioData));
        $m = array_values(unserialize($this->ratioData));
        $sum[0] = $m[0];
        for ($l = 1; $l < count($m); $l++) $sum[$l] = $sum[$l - 1] + $m[$l];
        for ($x = $this->a->getX(); $x <= $this->b->getX(); $x++) {
            for ($y = $this->a->getY(); $y <= $this->b->getY(); $y++) {
                for ($z = $this->a->getZ(); $z <= $this->b->getZ(); $z++) {
                    $a = rand(0, end($sum));
                    for ($l = 0; $l < count($sum); $l++) {
                        if ($a <= $sum[$l]) {
                            $hash = Level::chunkHash($x >> 4, $z >> 4);
                            if(isset($chunks[$hash])) $chunks[$hash]->setBlock($x & 0x0f, $y & 0x7f, $z & 0x0f, $id[$l] & 0xff, 0);
                            $l = count($sum);
                        }
                    }
                }
            }
        }
        $this->setResult($chunks);
    }
    public function onCompletion(Server $server){
        $chunks = $this->getResult();
        $plugin = $server->getPluginManager()->getPlugin("MineReset");
        if($plugin instanceof MineReset and $plugin->isEnabled()) {
            $level = $server->getLevel($this->levelId);
            if ($level != null) {
                foreach ($chunks as $hash => $chunk) {
                    Level::getXZ($hash, $x, $z);
                    $level->setChunk($x, $z, $chunk);
                }
            }
        }
    }
}
```

### To conclude
It's fine if you choose to use an async or a tick separated block set (or even a standard one). The important thing is that we pay attention to major bottlenecks (like this one) and improve our plugins.



