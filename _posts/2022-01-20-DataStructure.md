# Data Structure
- SkipList
```
type Node struct {
	key, val int
	forward []*Node
}

type SkipList struct {
	head *Node
	maxLevel int
}

func NewSkipList(maxLevel int) *SkipList {
	if maxLevel <= 0 {
		maxLevel = 1
	}
	return &SkipList{
		head: &Node{
			key: -1,
			val: -1,
			forward: make([]*Node, maxLevel),
		},
		maxLevel: maxLevel,
	}
}

func (sl *SkipList) Search(key int) int {
	cur := sl.head
	for i := sl.maxLevel - 1; i >= 0; i-- {
		for cur.forward[i] != nil && cur.forward[i].key < key {
			cur = cur.forward[i]
		}
	}

	if cur.forward[0] != nil {
		cur = cur.forward[0]
	}

	if cur.key == key {
		return cur.val
	}

	return -1
}

func (sl *SkipList) Insert(key, val int) {
	cur := sl.head
	update := make([]*Node, sl.maxLevel)
	for i := sl.maxLevel - 1; i >= 0; i-- {
		if cur.forward[i] != nil && cur.forward[i].key < key {
			cur = cur.forward[i]
		}
		update[i] = cur
	}

	if cur.forward[0] != nil && cur.forward[0].key == key {
		cur = cur.forward[0]
		cur.val = val
		return
	}

	v := sl.randomLevel()
	if v > sl.maxLevel {
		for i := sl.maxLevel; i < v; i++ {
			update = append(update, sl.head)
		}
		sl.maxLevel = v
	}
	x := &Node{
		key: key,
		val: val,
		forward: make([]*Node, v),
	}

	for i := 0; i < v; i++ {
		x.forward[i] = update[i].forward[i]
		update[i].forward[i] = x
	}
}

func (sl *SkipList) randomLevel() int {
	v := 1
	cur := float32(rand.Intn(10))/10
	for cur < 0.5 && v < sl.maxLevel {
		v += 1
		cur = float32(rand.Intn(10))/10
	}
	return v
}

func(sl *SkipList) Delete(key int) {
	cur := sl.head
	update := make([]*Node, sl.maxLevel)
	for i := sl.maxLevel - 1; i >= 0; i-- {
		for cur.forward[i] != nil && cur.forward[i].key < key {
			cur = cur.forward[i]
		}
		update[i] = cur
	}

	if cur.forward[0] == nil {
		return
	}

	cur = cur.forward[0]
	if cur.key == key {
		for i := 0; i < len(update); i++ {
			if update[i].forward[i] != cur {
				break
			}
			update[i].forward[i] = cur.forward[i]
		}

		for i := sl.maxLevel - 1; i >= 0; i-- {
			if sl.head.forward[i] == nil {
				sl.maxLevel -= 1
			}
		}
		sl.head.forward = sl.head.forward[:sl.maxLevel]
	}
}
```