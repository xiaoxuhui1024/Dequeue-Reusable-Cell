# Dequeue-Reusable-Cell
UITableViewCell的重用机制

##### UITableView继承自UIScrollview,是苹果为我们封装好的一个基于scroll的控件。上面主要是一个个的UITableViewCell,可以让UITableViewCell响应一些点击事件，也可以在UITableViewCell中加入UITextField或者UITextView等子视图，使得可以在cell上进行文字编辑。

##### UITableView中的cell可以有很多，一般会通过重用cell来达到节省内存的目的:通过为每个cell指定一个重用标识符(reuseIdentifier),即指定了单元格的种类,当cell滚出屏幕时,会将滚出屏幕的单元格放入重用的queue中，当某个未在屏幕上的单元格要显示的时候，就从这个queue中取出单元格进行重用。

##### 但对于多变的自定义cell，有时这种重用机制会出错。比如，当一个cell含有一个UITextField的子类并被放在重用queue中以待重用，这时如果一个未包含任何子视图的cell要显示在屏幕上，就会取出并使用这个重用的cell显示在无任何子视图的cell中，这时候就会出错。


### 解决方法

- 方法1

将获得cell的方法从- (UITableViewCell*)dequeueReusableCellWithIdentifier:(NSString*)identifier 换为-(UITableViewCell *)cellForRowAtIndexPath:(NSIndexPath *)indexPath

重用机制调用的就是dequeueReusableCellWithIdentifier这个方法，方法的意思就是“出列可重用的cell”，因而只要将它换为cellForRowAtIndexPath（只从要更新的cell的那一行取出cell），就可以不使用重用机制，因而问题就可以得到解决，虽然可能会浪费一些空间

```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
static NSString *CellIdentifier = @"Cell";
// UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:CellIdentifier]; //改为以下的方法
UITableViewCell *cell = [tableView cellForRowAtIndexPath:indexPath]; //根据indexPath准确地取出一行，而不是从cell重用队列中取出
if (cell == nil) {
cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:CellIdentifier];
}
//...其他代码
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
static NSString *CellIdentifier = @"Cell";
// UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:CellIdentifier]; //改为以下的方法
UITableViewCell *cell = [tableView cellForRowAtIndexPath:indexPath]; //根据indexPath准确地取出一行，而不是从cell重用队列中取出
if (cell == nil) {
cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:CellIdentifier];
}
//...其他代码
}
```

- 方法2
通过为每个cell指定不同的重用标识符(reuseIdentifier)来解决。
重用机制是根据相同的标识符来重用cell的，标识符不同的cell不能彼此重用。于是我们将每个cell的标识符都设置为不同，就可以避免不同cell重用的问题了。
```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{

NSString *CellIdentifier = [NSString stringWithFormat:@"Cell%d%d", [indexPath section], [indexPath row]];//以indexPath来唯一确定cell
UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:CellIdentifier]; //出列可重用的cell
if (cell == nil) {
cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:CellIdentifier];
}
//...其他代码
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{

NSString *CellIdentifier = [NSString stringWithFormat:@"Cell%d%d", [indexPath section], [indexPath row]];//以indexPath来唯一确定cell
UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:CellIdentifier]; //出列可重用的cell
if (cell == nil) {
cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:CellIdentifier];
}
//...其他代码
}
```

- 方法3
删除重用cell的所有子视图 从而得到一个没有特殊格式的cell，供其他cell重用。
```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
static NSString *CellIdentifier = @"Cell";
UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:CellIdentifier]; //出列可重用的cell
if (cell == nil) {
cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:CellIdentifier];
}
else
{
//删除cell的所有子视图
while ([cell.contentView.subviews lastObject] != nil)
{
[(UIView*)[cell.contentView.subviews lastObject] removeFromSuperview];
}
}
//...其他代码
}
```

