![](https://tcs.teambition.net/storage/312hde2941131198cf24f73c8e2986169668?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQyOCwiaWF0IjoxNjcxMjQ2NjI4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhkZTI5NDExMzExOThjZjI0ZjczYzhlMjk4NjE2OTY2OCJ9.7mo8V0yNwYRuY_0XsR1ZJmiN-RBwBWWpGzzexQptmTE&download=image.png "")

![](https://tcs.teambition.net/storage/312hd201899597c2d7793b3c1ba74ce8d3bd?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQyOCwiaWF0IjoxNjcxMjQ2NjI4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhkMjAxODk5NTk3YzJkNzc5M2IzYzFiYTc0Y2U4ZDNiZCJ9.4Oihtbj7KL0rjt43ng1omWinPhH1ldqH-DLszzZ1bPE&download=image.png "")

# 传统方法

```python
def get_publish_times(pub_topic_name: str) -> List[pd.Timestamp]:
    """Get publication timestamps for topic."""
    # Get all publish instances
    pub_instances = data_util.get_publish_instances()

    # Get publisher handle
    pub_handle = get_handle('pub', pub_topic_name)

    # Since publish calls go rclcpp->rcl->rmw and since we
    # only know the publisher handle at the rcl level, we first
    # get the indexes of all rcl_publish calls for our publisher
    rcl_pub_indexes = pub_instances.loc[
        pub_instances['publisher_handle'] == pub_handle
    ].index.values.astype(int)
    rcl_pub_indexes = list(rcl_pub_indexes)
    max_index = pub_instances.index.values.astype(int).max()
    # Then we group rclcpp & rmw calls (before & after,
    # respectively) with matching message pointers
    rclcpp_rmw_pub_timestamps = []
    for rcl_pub_index in rcl_pub_indexes:
        # Get message pointer value
        message = pub_instances.iloc[rcl_pub_index]['message']
        # Get corresponding rclcpp_publish row
        rclcpp_timestamp = None
        rclcpp_pub_index = rcl_pub_index - 1
        while rclcpp_pub_index >= 0:
            # Find the first row above with the same message
            row = pub_instances.iloc[rclcpp_pub_index]
            if message == row['message'] and 'rclcpp' == row['layer']:
                rclcpp_timestamp = row['timestamp']
                break
            rclcpp_pub_index -= 1
        # Get corresponding rmw_publish row
        rmw_timestamp = None
        rmw_pub_index = rcl_pub_index + 1
        while rmw_pub_index <= max_index:
            # Find the first row below rcl_publish row with the same message
            row = pub_instances.iloc[rmw_pub_index]
            if message == row['message'] and 'rmw' == row['layer']:
                rmw_timestamp = row['timestamp']
                break
            rmw_pub_index += 1

        assert rclcpp_timestamp is not None and rmw_timestamp is not None
        rclcpp_rmw_pub_timestamps.append(rclcpp_timestamp + (rmw_timestamp - rclcpp_timestamp) / 2)

    return rclcpp_rmw_pub_timestamps

```

# e2e方法

e2e方法好像不太支持
