/* ID OF QUEST YOU WANT TO COMPLETE */
let questId = "1227767407154561034"
/* MINUTES OF STREAMING REQUIRED */
let minutes = 15

/* this snippet is a remaster of seif_2009's snippet */

let sleep = ms => new Promise(resolve => setTimeout(resolve, ms));

function updateProgress(number) {
try {
    let parentElement = document.querySelector('.progress-quest');
    let childElement = parentElement.children[0];

    let percent = -100 + number
    childElement.style = `transform: translate3d(${percent}%, 0px, 0px); background-color: var(--brand-500);`
} catch(e){}

}

function questSnippet() {
    if (typeof findByProps === 'undefined') {
        console.warn("This snippet requires FindByProps, please paste it before running this snippet")
        console.log("To run this snippet again, just type questSnippet()")
        return;
    }
    const comp = findByProps("openModal", "ConfirmModal")
    const jsx = findByProps("jsx").jsx
    comp.openModal(props =>
        jsx(comp.ConfirmModal, {
            header: "Before you continue...",
            confirmText: "Yes, I match requirements",
            cancelText: "Ok, lemme do it",
            confirmButtonColor: comp.Button.Colors.BRAND,
            onConfirm: () => {

                if (!findByProps("getStreamerActiveStreamMetadata").getCurrentUserActiveStream()) {
                    findByProps("showToast").showToast(
                        findByProps("createToast").createToast("Please match requirements", findByProps("ToastType").ToastType.FAILURE, {
                            duration: 8000
                        })
                    )
                }
                let streamId = findByProps("encodeStreamKey").encodeStreamKey(findByProps("getStreamerActiveStreamMetadata").getCurrentUserActiveStream())

                let modalId = comp.openModal(props =>
                    jsx(comp.Dialog, {
                        style: {
                            display: "flex",
                            flexDirection: "column",
                            alignItems: "center",
                            justifyContent: "center"
                        },
                        children: [jsx("img", {
                            class: findByProps("img").img,
                            src: "https://cdn.discordapp.com/badge-icons/7d9ae358c8c5e118768335dbe68b4fb8.png",
                            style: {
                                width: "150px",
                                height: "150px"
                            },
                        }), jsx(comp.Heading, {
                            style: {
                                padding: "30px 140px"
                            },
                            class: findByProps("modalTitle").modalTitle,
                            children: "Completing Quest"
                        }), jsx(comp.Heading, {
                            style: {
                                marginBottom: "10px"
                            },
                            variant: "text-s/normal",
                            children: "While you may close this popup, make sure not to leave the stream, close or refresh Discord during this process!"
                        }), jsx(comp.Progress, {
                            className: "progress-quest",
                            animate: true,
                            percent: 0,
                        })],
                    })
                );
                let heartbeat = async function() {
                    let progress = 0;

                    while (progress < (minutes * 60)) {
                        let res = await findByProps("getAPIBaseURL").HTTP.post({
                            url: `/quests/${questId}/heartbeat`,
                            body: {
                                stream_key: streamId
                            }
                        });

                        if (progress === 0) {
                            progress = res.body.stream_progress_seconds;
                        }

                        if ((100 * progress) / (minutes * 60) >= 100) {
                            finishQuest(modalId, "Quest already completed!", "SUCCESS")
                            return;
                        }

                        let count = 0;
                        const interval = setInterval(() => {
                            count++;
                            progress++;
                            let percentComplete = (100 * progress) / (minutes * 60);
                            console.log(percentComplete)
                            updateProgress(percentComplete)
                            console.log('Count:', count, 'Progress:', progress, 'Percent Complete:', percentComplete.toFixed(2) + '%');

                            if (count >= 30) {
                                clearInterval(interval);
                            }
                        }, 1000);
                        await new Promise(resolve => setTimeout(resolve, 30000));
                    }

                    finishQuest(modalId, "Quest completed!", "SUCCESS")
                };

                heartbeat();
            },
            onCancel: () => {
                console.log("To restart this snippet, type questSnippet() in console");
            },
            ...props,
            children: jsx(comp.Text, {
                variant: "text-md/bold",
                children: ["Make sure :", jsx(comp.Text, {
                    variant: "text-md/normal",
                    children: ["- You accepted the quest", jsx(comp.Text, {
                        variant: "text-md/normal",
                        children: ["- You joined a voice channel with a friend/alt", jsx(comp.Text, {
                            variant: "text-md/normal",
                            children: ["- You are streaming something there (can be anything, but don't stop streaming until quest is completed)", jsx(comp.Text, {
                                variant: "text-xs/normal",
                                style: {
                                    marginTop: "10px",
                                },
                                children: ["To restart this snippet, type questSnippet() in console", ]
                            })]
                        })]
                    })]
                })]
            }),
        })
    );

    function finishQuest(modalId, text, type) {
        comp.closeModal(modalId)

        if(type === "SUCCESS"){
        findByProps("showToast").showToast(
            findByProps("createToast").createToast(text, findByProps("ToastType").ToastType.SUCCESS, {
                duration: 8000
            })
        );
        } else {
             findByProps("showToast").showToast(
            findByProps("createToast").createToast(text, findByProps("ToastType").ToastType.FAILURE, {
                duration: 8000
            })
        );
        }
    }
}
questSnippet()
