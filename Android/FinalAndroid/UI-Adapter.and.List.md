
`getView()`的第二个参数，一般叫`convertView`。它可能是null。但如果不为null，则它实际是我们之前创建过的一个View。这种情况主要发生在用户滚动了列表。As new rows appear, Android will attempt to recycle the views of the rows that scrolled off the other end of the list, to save us from having to rebuild them from scratch. The advantage is that we avoid the potentially expensive inflation step. In fact, according to statistics cited by Google at the 2010 Google I|O conference, a ListView that uses a recycling `ListAdapter` will perform 150 percent faster than one that does not.




