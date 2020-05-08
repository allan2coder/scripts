```
/*
 * @Description: 
 * @Autor: allan
 * @Date: 2020-05-07 15:29:23
 * @LastEditors: allan
 * @LastEditTime: 2020-05-08 10:01:59
 */
const UID = '5ea808b2e51d454d894405f3';
const TOPIC = '5eb36a906fb9a05a2169ff7c';
const X_JUNJIN_CLIENT = '1588815198735';
const X_JUNJIN_TOKEN = 'eyJhY2Nlc3NfdG9rZW4iOiJUZEhwTGpoWGdyYTI2em5KIiwicmVmcmVzaF90b2tlbiI6ImFoT3hnc1IxRktJa0tTUkYiLCJ0b2tlbl90eXBlIjoibWFjIiwiZXhwaXJlX2luIjoyNTkyMDAwfQ==';
const COUNT = 3;

const HOT_TOPIC_COMMENT_URL = `https://hot-topic-comment-wrapper-ms.juejin.im/v1/comments/${TOPIC}`;
const FOLLOWER_URL = 'https://follow-api-ms.juejin.im/v1/getUserFollowerList';

/**
 * Fisher–Yates shuffle
 */
Array.prototype.shuffle = function () {
    const input = this;

    for (let i = input.length - 1; i >= 0; i--) {
        const randomIndex = Math.floor(Math.random() * (i + 1));
        const itemAtIndex = input[randomIndex];
        input[randomIndex] = input[i];
        input[i] = itemAtIndex;
    }
    return input;
};

/**
 * 获取沸点评论
 */
async function getAllComments(pageNum = 1) {
    const { d: { comments } } = await fetch(`${HOT_TOPIC_COMMENT_URL}?pageNum=${pageNum}&pageSize=20`, {
        headers: {
            'x-juejin-client': X_JUNJIN_CLIENT,
            'x-juejin-token': X_JUNJIN_TOKEN,
            'x-juejin-src': 'web',
        },
    }).then(res => res.json());
    if (comments.length === 0) return [];
    const others = await getAllComments(pageNum + 1);
    return [...comments, ...others];
}

/**
 * 获取关注者
 */
async function getAllFollowers(before = '') {
    const { d: followers } = await fetch(`${FOLLOWER_URL}?uid=${UID}&currentUid=${UID}&before=${before}&src=web`).then(res => res.json());
    if (followers.length === 0) return [];
    const others = await getAllFollowers(followers.reverse()[0].createdAtString);
    return [...followers, ...others];
}


(async () => {
    const t = new Date();
    console.log(`当前时间：${t.toLocaleDateString()} ${t.toLocaleTimeString()}`);

    console.log('正在获取沸点评论……');
    const comments = await getAllComments();
    const commentUsers = comments.map(comment => comment.userInfo.username);

    console.log('正在获取关注者……');
    const followers = await getAllFollowers();
    const followerUsers = followers.map(follow => follow.follower.username);

    const validUsers = Array.from(new Set(commentUsers)).filter(user => followerUsers.includes(user));
    console.log('有效参与抽奖用户', validUsers);

    const luckyUsers = validUsers.shuffle().slice(0, COUNT);
    console.log('中奖用户', luckyUsers);
})();
```
