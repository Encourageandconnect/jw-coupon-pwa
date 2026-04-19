import React from 'react'

export default function App() {
  const couponCategories = {
    'Hospitality activities': [
      {
        title: 'Meal After Meeting',
        verse: 'Romans 12:13',
        idea: 'Invite someone for a simple meal and warm association.'
      },
      {
        title: 'Encouragement Visit',
        verse: '1 Thessalonians 5:11',
        idea: 'Visit someone who may appreciate encouragement.'
      },
      {
        title: 'Coffee and Conversation',
        verse: 'Hebrews 10:24, 25',
        idea: 'Set aside time for uplifting conversation.'
      }
    ],
    'Gathering activities': [
      {
        title: 'Family Game Night',
        verse: 'Psalm 133:1',
        idea: 'Enjoy wholesome association with families and friends.'
      },
      {
        title: 'Picnic Fellowship',
        verse: 'Romans 15:2',
        idea: 'Organize a simple outdoor gathering.'
      },
      {
        title: 'Encouragement Circle',
        verse: 'Proverbs 12:25',
        idea: 'Gather to share positive scriptures and encouragement.'
      }
    ],
    'Ministry activities': [
      {
        title: 'Service Morning',
        verse: 'Matthew 6:33',
        idea: 'Invite others to share in the ministry together.'
      },
      {
        title: 'Return Visit Day',
        verse: '2 Timothy 4:5',
        idea: 'Set time aside for return visits and follow-up.'
      },
      {
        title: 'Letter Writing Evening',
        verse: 'Galatians 6:10',
        idea: 'Write encouraging letters as a group.'
      }
    ],
    'Summer activities': [
      {
        title: 'Park Day Invite',
        verse: 'Ecclesiastes 3:13',
        idea: 'Enjoy a refreshing summer outing together.'
      },
      {
        title: 'Outdoor Hospitality',
        verse: '1 Peter 4:9',
        idea: 'Host a backyard meal or fellowship event.'
      },
      {
        title: 'Summer Encouragement Walk',
        verse: 'Psalm 55:22',
        idea: 'Take a walk and offer encouragement to someone.'
      }
    ],
    'Spring activities': [
      {
        title: 'Spring Visit Coupon',
        verse: 'Galatians 6:2',
        idea: 'Offer practical help or a cheerful visit.'
      },
      {
        title: 'Garden Fellowship',
        verse: 'Genesis 2:15',
        idea: 'Spend time outdoors with others in a peaceful setting.'
      },
      {
        title: 'Spring Refreshment Invite',
        verse: 'Isaiah 40:31',
        idea: 'Plan a light seasonal gathering.'
      }
    ],
    Other: [
      {
        title: 'Custom Kindness Coupon',
        verse: '2 Corinthians 9:7',
        idea: 'Create your own act of generosity.'
      },
      {
        title: 'Practical Help Day',
        verse: 'Philippians 2:4',
        idea: 'Offer support with errands or household tasks.'
      },
      {
        title: 'Special Encouragement Note',
        verse: 'Proverbs 16:24',
        idea: 'Give a handwritten note or thoughtful message.'
      }
    ]
  }

  const categories = Object.keys(couponCategories)
  const [selectedCategory, setSelectedCategory] = React.useState(categories[0])
  const [selectedCoupon, setSelectedCoupon] = React.useState(
    couponCategories[categories[0]][0]
  )
  const [customTitle, setCustomTitle] = React.useState('')
  const [date, setDate] = React.useState('')
  const [time, setTime] = React.useState('')
  const [invitees, setInvitees] = React.useState('')
  const [message, setMessage] = React.useState(
    'You are warmly invited to join this meaningful activity.'
  )
  const [created, setCreated] = React.useState(false)
  const [notificationStatus, setNotificationStatus] = React.useState('Not enabled')
  const [deferredPrompt, setDeferredPrompt] = React.useState(null)
  const [installReady, setInstallReady] = React.useState(false)

  const currentCoupons = couponCategories[selectedCategory]
  const finalTitle = customTitle.trim() || selectedCoupon.title

  React.useEffect(() => {
    setSelectedCoupon(couponCategories[selectedCategory][0])
  }, [selectedCategory])

  React.useEffect(() => {
    const handleBeforeInstallPrompt = (e) => {
      e.preventDefault()
      setDeferredPrompt(e)
      setInstallReady(true)
    }

    window.addEventListener('beforeinstallprompt', handleBeforeInstallPrompt)
    return () =>
      window.removeEventListener('beforeinstallprompt', handleBeforeInstallPrompt)
  }, [])

  const inviteText = `Encourage & Connect Invitation

Category: ${selectedCategory}
Activity: ${finalTitle}
Scripture: ${selectedCoupon.verse}
Idea: ${selectedCoupon.idea}
When: ${date || 'Not set'} ${time || ''}
Invitees: ${invitees || 'None added'}

Message:
${message}`

  const copyInvite = async () => {
    try {
      await navigator.clipboard.writeText(inviteText)
      alert('Invitation copied.')
    } catch {
      alert('Copy failed. Please copy it manually from the preview area.')
    }
  }

  const installApp = async () => {
    if (!deferredPrompt) return
    deferredPrompt.prompt()
    await deferredPrompt.userChoice
    setDeferredPrompt(null)
    setInstallReady(false)
  }

  const downloadICS = () => {
    if (!date || !time) {
      alert('Please choose a date and time first.')
      return
    }

    const start = new Date(`${date}T${time}`)
    const end = new Date(start.getTime() + 60 * 60 * 1000)

    const formatDate = (d) =>
      d.toISOString().replace(/[-:]/g, '').split('.')[0] + 'Z'

    const ics = `BEGIN:VCALENDAR
VERSION:2.0
BEGIN:VEVENT
SUMMARY:${finalTitle}
DESCRIPTION:${message}
DTSTART:${formatDate(start)}
DTEND:${formatDate(end)}
END:VEVENT
END:VCALENDAR`

    const blob = new Blob([ics], { type: 'text/calendar' })
    const url = URL.createObjectURL(blob)
    const a = document.createElement('a')
    a.href = url
    a.download = 'encourage-connect-invitation.ics'
    a.click()
    URL.revokeObjectURL(url)
  }

  const requestPushNotifications = async () => {
    if (!('Notification' in window)) {
      setNotificationStatus('Notifications not supported')
      return
    }

    const permission = await Notification.requestPermission()

    if (permission === 'granted') {
      setNotificationStatus('Enabled')
      new Notification('Encourage & Connect Notifications Enabled', {
        body: `You will receive reminders for ${finalTitle}.`
      })
    } else {
      setNotificationStatus('Permission denied')
    }
  }

  const smsText = encodeURIComponent(inviteText)
  const smsLink = `sms:?body=${smsText}`

  return (
    <div className="min-h-screen bg-purple-50 p-6">
      <div className="mx-auto grid max-w-7xl gap-6 lg:grid-cols-[1.2fr,0.8fr]">
        <div className="rounded-3xl border border-purple-200 bg-white p-6 shadow-sm">
          <div className="flex flex-col gap-2">
            <h1 className="text-3xl font-semibold text-purple-900">
              Encourage & Connect
            </h1>
            <p className="text-sm text-purple-700">
              Simple invitations for meaningful connection
            </p>
          </div>

          <div className="mt-4 flex flex-wrap gap-3">
            {installReady && (
              <button
                onClick={installApp}
                className="rounded-2xl bg-purple-700 px-5 py-3 text-sm font-medium text-white hover:bg-purple-800"
              >
                Install App
              </button>
            )}

            <button
              onClick={requestPushNotifications}
              className="rounded-2xl border border-purple-300 px-5 py-3 text-sm font-medium text-purple-900"
            >
              Enable Notifications
            </button>
          </div>

          <div className="mt-6">
            <label className="mb-2 block text-sm font-medium text-purple-900">
              Category
            </label>
            <div className="flex flex-wrap gap-2">
              {categories.map((category) => (
                <button
                  key={category}
                  onClick={() => setSelectedCategory(category)}
                  className={`rounded-full border px-4 py-2 text-sm ${
                    selectedCategory === category
                      ? 'border-purple-700 bg-purple-700 text-white'
                      : 'border-purple-300 bg-white text-purple-900'
                  }`}
                >
                  {category}
                </button>
              ))}
            </div>
          </div>

          <div className="mt-6 grid gap-4 md:grid-cols-2 xl:grid-cols-3">
            {currentCoupons.map((coupon) => (
              <button
                key={coupon.title}
                onClick={() => setSelectedCoupon(coupon)}
                className={`rounded-3xl border p-5 text-left shadow-sm transition ${
                  selectedCoupon.title === coupon.title
                    ? 'border-purple-700 bg-purple-100'
                    : 'border-purple-200 bg-white hover:bg-purple-50'
                }`}
              >
                <div className="text-xs uppercase tracking-wide text-purple-500">
                  Coupon
                </div>
                <div className="mt-2 text-lg font-semibold text-purple-900">
                  {coupon.title}
                </div>
                <div className="mt-2 text-sm text-purple-700">{coupon.idea}</div>
                <div className="mt-4 inline-block rounded-full border border-amber-300 bg-amber-50 px-3 py-1 text-xs text-amber-700">
                  {coupon.verse}
                </div>
              </button>
            ))}
          </div>

          <div className="mt-8 grid gap-4">
            <input
              value={customTitle}
              onChange={(e) => setCustomTitle(e.target.value)}
              placeholder="Custom coupon title"
              className="w-full rounded-2xl border border-purple-300 px-4 py-3"
            />

            <div className="grid gap-4 sm:grid-cols-2">
              <input
                type="date"
                value={date}
                onChange={(e) => setDate(e.target.value)}
                className="rounded-2xl border border-purple-300 px-4 py-3"
              />
              <input
                type="time"
                value={time}
                onChange={(e) => setTime(e.target.value)}
                className="rounded-2xl border border-purple-300 px-4 py-3"
              />
            </div>

            <input
              value={invitees}
              onChange={(e) => setInvitees(e.target.value)}
              placeholder="Invitees"
              className="w-full rounded-2xl border border-purple-300 px-4 py-3"
            />

            <textarea
              value={message}
              onChange={(e) => setMessage(e.target.value)}
              rows={5}
              className="w-full rounded-2xl border border-purple-300 px-4 py-3"
            />

            <div className="flex flex-wrap gap-3">
              <button
                onClick={() => setCreated(true)}
                className="rounded-2xl bg-purple-700 px-5 py-3 text-sm font-medium text-white hover:bg-purple-800"
              >
                Create Coupon
              </button>

              <button
                onClick={copyInvite}
                className="rounded-2xl border border-purple-300 px-5 py-3 text-sm font-medium text-purple-900"
              >
                Copy Invite
              </button>

              <button
                onClick={downloadICS}
                className="rounded-2xl border border-purple-300 px-5 py-3 text-sm font-medium text-purple-900"
              >
                Add to Calendar
              </button>

              <a
                href={smsLink}
                className="rounded-2xl border border-purple-300 px-5 py-3 text-sm font-medium text-purple-900"
              >
                Send SMS
              </a>
            </div>

            <div className="rounded-2xl border border-purple-200 bg-purple-50 p-4 text-sm text-purple-700">
              Notification status: {notificationStatus}
            </div>
          </div>
        </div>

        <div className="rounded-3xl border border-purple-200 bg-white p-6 shadow-sm">
          <h2 className="text-2xl font-semibold text-purple-900">Coupon Preview</h2>

          <div className="mt-5 rounded-3xl border-2 border-dashed border-purple-300 bg-purple-50 p-6">
            <div className="text-xs uppercase tracking-[0.25em] text-purple-500">
              Encourage & Connect Coupon
            </div>

            <div className="mt-3 text-2xl font-semibold text-purple-900">
              {finalTitle}
            </div>

            <div className="mt-2 inline-block rounded-full border border-amber-300 bg-amber-50 px-3 py-1 text-xs text-amber-700">
              {selectedCategory}
            </div>

            <div className="mt-3 text-sm text-purple-700">
              {selectedCoupon.verse}
            </div>

            <div className="mt-4 text-sm text-purple-900">
              {selectedCoupon.idea}
            </div>

            <div className="mt-5 grid gap-3 sm:grid-cols-2">
              <div className="rounded-2xl border border-purple-200 bg-white p-3">
                <span className="text-xs text-purple-500">Date</span>
                <div className="mt-1 text-sm font-medium text-purple-900">
                  {date || 'Not set'}
                </div>
              </div>

              <div className="rounded-2xl border border-purple-200 bg-white p-3">
                <span className="text-xs text-purple-500">Time</span>
                <div className="mt-1 text-sm font-medium text-purple-900">
                  {time || 'Not set'}
                </div>
              </div>
            </div>

            <div className="mt-3 rounded-2xl border border-purple-200 bg-white p-3">
              <span className="text-xs text-purple-500">Invitees</span>
              <div className="mt-1 text-sm text-purple-900">
                {invitees || 'None added yet'}
              </div>
            </div>

            <div className="mt-3 rounded-2xl border border-purple-200 bg-white p-3">
              <span className="text-xs text-purple-500">Message</span>
              <div className="mt-2 whitespace-pre-wrap text-sm text-purple-900">
                {message}
              </div>
            </div>
          </div>

          {created && (
            <div className="mt-4 rounded-2xl border border-green-200 bg-green-50 p-4 text-sm text-green-800">
              Invitation created successfully.
            </div>
          )}

          <div className="mt-5">
            <label className="mb-2 block text-sm font-medium text-purple-900">
              Shareable invitation text
            </label>
            <textarea
              readOnly
              value={inviteText}
              rows={12}
              className="w-full rounded-2xl border border-purple-300 p-4 text-sm"
            />
          </div>
        </div>
      </div>
    </div>
  )
}